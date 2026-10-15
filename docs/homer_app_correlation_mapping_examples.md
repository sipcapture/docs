Let's say you have SIP CALLS, and all SIP messages have been stored in the table **hep_proto_1_call**, and your LOGs are stored in **hep_proto_100_logs**

For protocol `HEP ID: 1`, `Profile: Call`, you can make an additional correlation MAPPING. 
Click to EDIT (the blue wrench in the following picture):

![Screenshot from 2019-05-20 13-59-14](https://user-images.githubusercontent.com/4513061/58020275-f8c29500-7b07-11e9-9def-ab286d07da4e.png)



And in the MAPPING, add your custom logic.

In this case, to correlate SIP CALLs traffic to your LOGs which are stored in **hep_proto_100_logs**:
we extract **callid** from the JSON body of **hep_proto_1_call** (below: `"source_field": "data_header.callid",`) and do the lookup to HEP: **100** (logs), profile: **default**
in destination field: **sid**, in the time-range (original) from+=-300, to+=200.

```javascript
[
  {
    "source_field": "data_header.callid",
    "lookup_id": 100,
    "lookup_profile": "default",
    "lookup_field": "sid",
    "lookup_range": [
      -300,
      200
    ]
  }
]
```

The SQL query will look like : 
```sql
select * from hep_proto_100_default where sid = 'CALLID';
```

## RTCP JSON correlation

Here is an example of how to do correlation to another protocol: RTCP JSON, HEP: 5 (below: `"lookup_id": 5,`), destination SID (below: `"lookup_field": "sid",`) can be any header from your JSON body. Here we choose `callid` i.e. `  "source_field": "data_header.callid",`:

```javascript
[  
  {
    "source_field": "data_header.callid",
    "lookup_id": 5,
    "lookup_profile": "default",
    "lookup_field": "sid",
    "lookup_range": [
      -300,
      200
    ]
  }
]
```

![Screenshot from 2019-05-20 14-03-36](https://user-images.githubusercontent.com/4513061/58020321-1b54ae00-7b08-11e9-9b13-b65e7d71437e.png)

and of course you can combine the mappings:
```javascript
[
  {
    "source_field": "data_header.callid",
    "lookup_id": 100,
    "lookup_profile": "default",
    "lookup_field": "sid",
    "lookup_range": [
      -300,
      200
    ]
  },
  {
    "source_field": "data_header.callid",
    "lookup_id": 5,
    "lookup_profile": "default",
    "lookup_field": "sid",
    "lookup_range": [
      -300,
      200
    ]
  }
]
```
## Correlation-ID correlation - graphing all calls with the same Correlation-ID ;)

For my SBC, it maps call-leg A, which has call-ID `123@abc` (it fills in the HEP packet correlation-id with `123`), and call-leg B which has call-ID `123@def` (whose HEP packet correlation-id has `123`). This means that the correlation-id for all SIP packets belonging to leg A and leg B (i.e. the same call, across the b2bua), are identical.

In the Settings -> mapping -> `SIP-1-call` profile, if you look in the lower `Fields Mapping` section below the `Correlation mapping` section, you may have a section like so, if you add Correlation ID as a search field in your SIP Calls panel (Usually called "Home"):
```javascript
...
    {
        "id": "protocol_header.correlation_id",
        "name": "Correlation ID",
        "type": "string",
        "index": "none",
        "form_type": "input",
        "position": 3,
        "skip": false,
        "hide": true,
        "sid_type": true
    },
...
```

This defines the Correlation ID column as a field to search for. If you want it there, go to your Call Search Settings, and set (click and drag) 'Correlation ID' field to active, then save. 

![Screenshot 2020-07-24 at 02 58 18](https://user-images.githubusercontent.com/647633/88352611-a22f9700-cd5a-11ea-81bb-bf7d62f3e497.png)

Then click the cog-wheel icon in your results panel (usually bottom left), and set 'Correlation ID' active, save. Now if you search for Correlation ID `123`, you should see only those two call legs A and B for `123` in the results, and the results should show the Correlation ID column.

![Screenshot 2020-07-24 at 03 08 55](https://user-images.githubusercontent.com/647633/88352771-41ed2500-cd5b-11ea-8403-005dbae5c8e6.png)



Now in the Settings -> mapping -> `SIP-1-call` profile, look in the upper `Correlation Mapping` section, add the following (i.e. replace existing content):

```javascript
[
    {
        "comment": "/*this graphs all calls with the same correlation-ID */",
        "source_field": "protocol_header.correlation_id",
        "lookup_id": 1,
        "lookup_profile": "call",
        "lookup_field": "protocol_header->>'correlation_id'",
        "lookup_range": [
            -300,
            200
        ]
    }
]
```
Note: ensure you don't have extra commas at line endings after `lookup_range`, or lack commas after each field line, otherwise the script doesn't run properly. You can safely copy-paste the above.

In the following image, you see the two resulting Call-ID shown at the bottom, when we click on a Call-ID/Correlation-ID displayed in the search results.

![Screenshot 2020-07-24 at 03 17 31](https://user-images.githubusercontent.com/647633/88353401-98f3f980-cd5d-11ea-811e-30e22cf6af5b.png)


If, like me, you want to also graph the RTCP packets on the call-graph, you can change Settings -> mapping -> `SIP-1-call` profile to the following:
```javascript
[
    {
        "comment": "/*this graphs RTCP for the respective call-ID*/",
        "source_field": "data_header.callid",
        "lookup_id": 5,
        "lookup_profile": "default",
        "lookup_field": "sid",
        "lookup_range": [
            -300,
            200
        ]
    },
    {
        "comment": "/*this graphs all calls with the same correlation-ID */",
        "source_field": "protocol_header.correlation_id",
        "lookup_id": 1,
        "lookup_profile": "call",
        "lookup_field": "protocol_header->>'correlation_id'",
        "lookup_range": [
            -300,
            200
        ]
    }
]
```

And your call graph should also show RTCP.

![Screenshot 2020-07-24 at 03 17 31](https://user-images.githubusercontent.com/647633/88353401-98f3f980-cd5d-11ea-811e-30e22cf6af5b.png)



## SIP-ISUP correlation.

In the below example, `input_function` will remove any leading `0` from the number and add it to the array.
`post_aggregation_field: sid` will aggregate the calls with the same `SID` (OPC:DPC:CIC)

So that the resulting SQL query might look like:
```sql
select * from hep_proto_54_default where data_header->'calling_number' IN ('0123456', '123456', '123456') and create_date BETWEEN '2019-02-02-XXXX' AND '2019-02-02-YYYYY'
```
and that the second SQL query might look like
```sql
select * from hep_proto_54_default where sid IN ('SID_FROM_LAST_QUERY' )
```

The following correlation mapping is necessary:

```javascript
  {
    "source_field": "data_header.from_user",
    "lookup_id": 54,
    "lookup_match_field": "data_header.method",
    "lookup_match_value": [
      "INVITE"
    ],
    "input_function": "data.forEach(function(el) {if(el.charAt(0) === '0') data.push(el.substr(1));});return data",
    "lookup_match_first": true,
    "lookup_profile": "default",
    "lookup_field": "data_header->>calling_number",
    "post_aggregation_field": "sid",
    "lookup_range": [
      -300,
      200
    ]
  },

```


### Remote Mapping
Correlation requests can be emitted to entities through the HEP pub-sub API, and dispatched by type. The following example will emit a data request to any entity providing `cdr` capabilities using the `source_field` specified in the mapping configuration:
```javascript
    {
      source_field: 'data_header.callid',
      lookup_id: 0,
      lookup_type: "pubsub",
      lookup_profile: 'cdr',
      lookup_field: '{"data":$source_field,"fromts":$fromts,"tots":$tots}',
      lookup_range: [-300, 200],
    }
```
