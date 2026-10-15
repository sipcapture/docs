This article describes the process and steps required to extract custom SIP `X-headers` for search and correlation purposes.

# 1. Add Custom Header in heplify-server 

`X-Headers` are extracted at the capture layer by `heplify-server` and converted to HEP header fields.

- *Correlation Only* Headers which do not require search should be _"normalized"_ to `correlation_id` headers using `AlegIDs`
- *Search + Correlation* Headers which require full indexing should be extracted using `CustomHeader` 

Edit `/etc/heplify-server.toml`

add custom header information 
```
AlegIDs               = ["XCallID"] # Custom Header(s) will be used as Correlation ID (first match)
ForceALegID           = true        # Enable search for A Leg Headers. default is false
CustomHeader          = ["X-CID","XCallID"] # Custom Headers will be Indexed for Header search
```

restart heplify-server
```
systemctl restart heplify-server.service
```
Now you will be able to see the custom `X-headers` in Packet details. Click on INVITE packet and then go to Details

![](https://i.imgur.com/ajngs6K.jpg)

# 2. Add Mappings in homer-app

Each header in SIP is mapped to a name in the schema to view/search in Homer UI, which is referred to as `Mapping`. For each custom header, we have to add new mapping accordingly.
  
Go to `Settings` in homer UI and click on `Mapping`, Edit **profile** 'call' with **HEP ID** '1'. 

![](https://i.imgur.com/w8DvAXQ.jpg)

There will be two kinds of `Mapping` 
1. **Correlation Mapping:** Defines any cross-protocol correlation mechanism
2. **Fields Mapping:** Defines schema and mapping for protocol fields

Go to `Fields Mapping`

![](https://i.imgur.com/9Xvtcts.jpg)

Add the following at end of `Fields Mapping`

```json
    {
        "id": "data_header.X-CID",
        "name": "X-CID",
        "type": "string",
        "index": "none",
        "form_type": "input",
        "position": 25,
        "skip": false,
        "hide": true
    },
    {
        "id": "data_header.XCallID",
        "name": "XCallID",
        "type": "string",
        "index": "none",
        "form_type": "input",
        "position": 26,
        "skip": false,
        "hide": true
    }
```

# 3. Call SIP Search Widget Settings

Go to `Home` screen of homer. Eidt settings of `Call SIP Search` widget. 

![](https://i.imgur.com/y1t6x4I.jpg)

Move X-CID and XCallID to active list. 

![](https://i.imgur.com/YcbD3on.jpg)

# 4. Search with Custom Header

now you will be able to search with a custom Header field.

![](https://i.imgur.com/6Vy6jYX.jpg) 

Note: Custom Header search will work with new calls only. 

----------

# Full Example for 2-way correlation

#### Mapping
```json
[
    {
       "source_field": "data_header.callid",
        "lookup_id": 1,
        "lookup_profile": "call",
        "append_sid": true,
        "lookup_field": "data_header->>'X-CID'",
        "lookup_range": [
            -300,
            200
        ]
    },
    {
        "source_field": "data_header.X-CID",
        "lookup_id": 1,
        "lookup_profile": "call",
        "append_sid": true,
        "lookup_field": "data_header->>'X-CID'",
        "lookup_range": [
            -300,
            200
        ]
    },
    {
        "source_field": "data_header.X-CID",
        "lookup_id": 1,
        "lookup_profile": "call",
        "append_sid": true,
        "lookup_field": "data_header->>'callid'",
        "lookup_range": [
            -300,
            200
        ]
    },
    {
        "source_field": "data_header.callid",
        "lookup_id": 1,
        "lookup_profile": "call",
        "lookup_field": "sid",
        "lookup_range": [
            -300,
            200
        ]
    }
]
```


### Field Modification
Fields can be modified and extended using the `input_function_js` functionality. The following example extends an extracted callid with `_b2b-1`

```json
 {
        "source_field": "data_header.callid",
        "lookup_id": 1,
        "lookup_profile": "call",
        "lookup_field": "sid",
        "input_function_js": "var returnData=[]; for (var i = 0; i < data.length; i++) { returnData.push(data[i]+'_b2b-1'); }; returnData;"
        "lookup_range": [
            -300,
            200
        ]
    }


```

Or multiple prefixes and suffixes `SBC-` , `_PBX-1`

```json
 {
        "source_field": "data_header.callid",
        "lookup_id": 1,
        "lookup_profile": "call",
        "lookup_field": "sid",
        "input_function_js": "var returnData=[]; for (var i = 0; i < data.length; i++) { returnData.push('SBC-'+data[i]); returnData.push(data[i]+'_PBX-1'); }; returnData;"
        "lookup_range": [
            -300,
            200
        ]
    }


```
