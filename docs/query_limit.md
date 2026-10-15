In a default (Docker assumed) installation of Homer7, the amount of results returned for SIP Call type searches is 200. 

To change this:

1. Edit your SIP Call search form
1. Add the `Query Limit` field to the `Active` column
1. Save the form
1. Enter e.g. `1000` in the `Query limit` field 
1. Search

Alternatively

1 - Settings
2 - User settings
3 - find your dashboard and edit the block
```javascript
...
                    {
                        "field_name": "limit",
                        "hepid": 1,
                        "name": "1:call:limit",
                        "selection": "Query Limit",
                        "type": "string",
                        "value": ""
                    },
...
```

to e.g. for 1000 results

```javascript
...
                    {
                        "field_name": "limit",
                        "hepid": 1,
                        "name": "1:call:limit",
                        "selection": "Query Limit",
                        "type": "string",
                        "value": "1000"
                    },
...
```
