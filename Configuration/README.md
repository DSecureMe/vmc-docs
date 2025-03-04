# Configuration

## Warning!
After installing the software and configuring all files as described in this documentation, verify their permissions. For example, the file /etc/vmc/config.yml may have default permissions rw-r--r--, due to the default behavior of umask in Linux. To enhance security, these permissions should be changed to rw------- (command: chmod 600 /etc/vmc/config.yml).

## Installation on a virtual machine
To be able to manage VMC from the level of systemctl, the following configuration files should be added to the directory `/etc/systemd/system`:

*vmc-admin.service*
```
[Unit]
Description=VMC Admin Panel

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vmcctl start admin

[Install]
WantedBy=multi-user.target
```

*vmc-scheduler.service*
```
[Unit]
Description=VMC Scheduler

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vmcctl start scheduler

[Install]
WantedBy=multi-user.target
```

*vmc-worker.service*
```
[Unit]
Description=VMC Worker

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vmcctl start worker

[Install]
WantedBy=multi-user.target
```

On each server where VMC is installed, a config.yml file will be created in the `/etc/vmc/` directory with the following structure:
*Sample VMC configuration file*
```
#VMC
vmc.ssl: True
vmc.domain: localhost
vmc.port: 443

#Redis (connection configuration)
redis.url: redis://:password@localhost:6379/1

#Elasticsearch
elasticsearch.hosts: ["http://127.0.0.1:9200"]
#elasticsearch.user: elastic
#elasticsearch.password: password

#database
database.engine: django.db.backends.postgresql_psycopg2
database.name: vmc
database.user: postgres
database.password: password
database.host: localhost
database.port: 5432

# Queue
rabbitmq.username: admin
rabbitmq.password: password
rabbitmq.host: localhost
rabbitmq.port: 5672

# Secret (it is recommended to change the default value)
secret_key: "random 52 characters, ex: jk&e^6%5@^5!^1jq8#vd2g^@8w9g5#i_v*ho!#mx%y7%5fuz9%"
#debug: true
```

On the server where the VMC Admin component is running, you must configure the proxy and serving static files:

*Sample proxy configuration for VMC Admin on Nginx server*
```
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    upstream vmc {
      server 127.0.0.1:8001;
    }

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;

     location / {
        proxy_pass http://vmc;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
      }

       location /static/ {
          alias /usr/share/vmc/static/;
       }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```

The following commands must be run on the server running VMC admin:
* Installation of static files necessary in the VMC admin panel:
```
vmcctl collectstatic
```
* Create a super admin user:
```
vmcctl createsuperuser
```
* Initialization of the core indexes in Elasticsearch:
```
vmcctl create_index
```

* Then enable and start the vmc-admin, vmc-scheduler and vmc-worker services:
```
systemctl enable vmc-admin
systemctl enable vmc-scheduler
systemctl enable vmc-worker

systemctl start vmc-admin
systemctl start vmc-scheduler
systemctl start vmc-worker
```

## Installation using the docker
Before starting the start-up of containers with VMC modules, make sure that all dependencies described in the Requirements chapter have been installed. Additional information on how to run VMC with dockers can be found in the [vmc-demo](https://github.com/DSecureMe/vmc-demo) repository.

For the components to work properly, the config.yml file should be prepared:
```
#VMC
vmc.ssl: True
vmc.domain: localhost
vmc.port: 443

#Redis (connection configuration)
redis.url: redis://:password@localhost:6379/1

#Elasticsearch
elasticsearch.hosts: ["http://127.0.0.1:9200"]
#elasticsearch.user: elastic
#elasticsearch.password: password

#database
database.engine: django.db.backends.postgresql_psycopg2
database.name: vmc
database.user: postgres
database.password: password
database.host: localhost
database.port: 5432

# Queue
rabbitmq.username: admin
rabbitmq.password: password
rabbitmq.host: localhost
rabbitmq.port: 5672

# Secret (it is recommended to change the default value)
secret_key: " random 52 characters, ex: jk&e^6%5@^5!^1jq8#vd2g^@8w9g5#i_v*ho!#mx%y7%5fuz9%"
#debug: true
```


### **Warning!**  
In the **docker-compose.yml** file, you can mount a file as a **read-only volume** in the container using the **`:ro`** syntax.  

### **Example of mounting a file as read-only in Docker Compose:**  

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    volumes:
      - ./config.yml:/etc/app/config.yml:ro
```

### **Explanation:**  
- **./config.yml** – the file located on the host  
- **/etc/app/config.yml** – the path where the file will be available inside the container  
- **`:ro`** – specifies that the file is mounted as read-only  

This ensures that the application inside the container can read the file but cannot modify it.
