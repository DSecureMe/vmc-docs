# API
To enable integration with other tools, VMC provides the REST API interface. Creating tokens allowing for correct authorization is described in chapter [Creation of API Tokens]().

## Downloading the connection configuration to the resource base
Endpoint allows you to download the configuration of the integrated resource base.

```
curl -H 'Authorization: Token <token>' http://<address vmc admin>/api/v1/assets-manager/config?name=<config name>'
```

Sample Response
```
{
    "name": "<config name>",
    "schema": "http",
    "host": "<ip address>",
    "port": 80,
    "username" :"<login>",
    "password": "<password>",
    "insecure": false,
    "enabled": true,
    "tenant": "<tenant name>"
}
```