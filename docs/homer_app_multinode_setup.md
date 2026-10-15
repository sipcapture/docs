If you want use Homer in the cluster mode, please open your webapp_config.json just add a new block:

```
 "database_data": {
    "node01": {
      "help": "Node #1 PGSQL Database (data)",
      "host": "10.0.0.1",
      "name": "homer_data",
      "node": "Node01",
      "pass": "homer_password",
      "user": "homer_user"
    },
    "node02": {
      "help": "Node #2 PGSQL Database (data)",
      "host": "10.0.0.2",
      "name": "homer_data",
      "node": "Node02",
      "pass": "homer_password",
      "user": "homer_user"
    }
  },

```

you can configure as many nodes as you want and in the UI you can select nodes where you want to search data
