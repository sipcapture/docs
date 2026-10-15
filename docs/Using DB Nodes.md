# HOMER 7 DB NODES
### Database Data
The data connections allow `homer-app` to communicate with one or more db nodes to fetch data results.
The `database_data` element can contain one or more objects reflecting each unique node:

##### Single Node
```
  "database_data": {
    "LocalNode": {
      "help": "Settings for PGSQL Database (data)",
      "node": "LocalNode",
      "user": "homer_user",
      "pass": "homer_password",
      "name": "homer_data",
      "host": "127.0.0.1"
    }
  }
```
##### Multi Node
```
  "database_data": {
    "LocalNode": {
      "help": "Settings for PGSQL Database (data)",
      "node": "LocalNode",
      "user": "homer_user",
      "pass": "homer_password",
      "name": "homer_data",
      "host": "127.0.0.1"
    },
    "RemoteNode": {
      "help": "Settings for PGSQL Database (data)",
      "node": "RemoteNode",
      "user": "homer_user",
      "pass": "homer_password",
      "name": "homer_data",
      "host": "127.0.0.2"
    }
  }
```
