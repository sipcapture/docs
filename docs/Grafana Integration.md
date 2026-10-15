When using Grafana on a different domain, HOMER offers an integrated proxy to bridge and authorize requests, embed widgets, etc:

```
"grafana_config": {
    "help": "Settings for Grafana",
    "host": "http://127.0.0.1:3000",
    "path": "/grafana",
    "proxy_control": false,
    "token": ""
}
```
