# API

To enable integration with other tools, VMC exposes a REST API. Tokens used
for authorisation are created in the admin panel — see
[Creation of API tokens](../Admin-Panel/README.md#creation-of-api-tokens).

## Downloading the asset-manager connection configuration

This endpoint returns the configuration of an integrated asset-management
source (Ralph).

```bash
curl -H 'Authorization: Token <token>' \
     'http://<vmc admin host>/api/v1/assets-manager/config?name=<config name>'
```

Sample response:

```json
{
    "name": "<config name>",
    "schema": "http",
    "host": "<ip address>",
    "port": 80,
    "username": "<login>",
    "password": "<password>",
    "insecure": false,
    "enabled": true,
    "tenant": "<tenant name>"
}
```

## Searching vulnerabilities

Looks up vulnerabilities for a given IP address inside a tenant scope.

```bash
curl -H 'Authorization: Token <token>' \
     'http://<vmc admin host>/api/v1/vulnerabilities?tenant=<tenant>&ip_address=<ip>'
```

The response is a list of vulnerability documents (see
[Vulnerability Collector](../Architecture/README.md#vulnerability-collector)
for the document schema).

## Downloading a raw scan file

Returns the original scanner report (XML) for the scan referenced by its
file ID (64-character SHA-256 hex).

```bash
curl -H 'Authorization: Token <token>' -o scan.xml \
     'http://<vmc admin host>/api/v1/scans/backups/<scan_file_id>'
```

The `<scan_file_id>` is taken from `scan_file_url` in the vulnerability
document.

## TheHive webhook

VMC accepts TheHive webhook events on:

```
http://<vmc admin host>/api/v1/webhook/thehive
```

See [Integration with TheHive — webhooks configuration](../HIVE-Integration/README.md#the-hive-webhooks-configuration)
for setup details.
