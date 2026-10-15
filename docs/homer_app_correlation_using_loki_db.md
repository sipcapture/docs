# How to make correlation using LOKI DB

`output_function`:

```javascript
const returnData = [];
for (let dt of data) {
    let arrayRet = {
        id: 0,
        sid: 'unknown',
        protocol_header: {
            captureId: 2001,
            capturePass: 'myHep',
            payloadType: 100,
            timeSeconds: 1548839273,
            timeUseconds: 649,
            protocolFamily: 2,
            method: 'LOKI'
        },
        data_header: {
            protocol: 17,
            dstIp: '127.0.0.1',
            srcIp: '127.0.0.1',
            dstPort: 0,
            srcPort: 0
        },
        raw: '',
        create_date: 0
    };
    arrayRet.id = dt.id;
    arrayRet.protocol_header.timeSeconds = new Date(dt.micro_ts).getTime() / 1000;
    arrayRet.protocol_header.timeUseconds = new Date(dt.micro_ts).getTime() % 1000;
    arrayRet.data_header.sid = 'unknown';
    arrayRet.data_header.srcPort = 1;
    arrayRet.data_header.dstPort = 2;
    arrayRet.data_header.srcIp = '127.0.0.2';
    arrayRet.data_header.payloadType = 200;
    arrayRet.data_header.create_date = dt.micro_ts;
    arrayRet.raw = dt.custom_1 + '\\r\\nTags:' + dt.custom_2;
    arrayRet.tag = dt.custom_2;
    arrayRet.create_date = new Date(dt.micro_ts).getTime();
    var n = dt.custom_1.search(' SrcIP=');
    if (n) {
        var ipData = dt.custom_1.substring(n + 1, dt.custom_1.length); /* split data */
        var entry = {};
        ipData.split(' ').forEach(function(pair) {
            var kv = pair.split('=');
            if (!kv[0] || !kv[1]) return;
            entry[kv[0]] = kv[1];
        });
        arrayRet.data_header.srcIp = entry['SrcIP'];
        arrayRet.data_header.srcPort = entry['SrcPort'];
        arrayRet.data_header.dstIp = entry['DstIP'];
        arrayRet.data_header.dstPort = entry['DstPort'];
        arrayRet.sid = entry['CID'];
        arrayRet.data_header.sid = entry['CID'];
    }
    var tagData = dt.custom_2.substring(1, dt.custom_2.length - 1);
    var tagArray = {};
    tagData.split(', ').forEach(function(pair) {
        var kv = pair.split('=');
        if (!kv[0] || !kv[1]) return;
        tagArray[kv[0]] = kv[1].replace(/\"/g, '');
    });
    if (tagArray.type) {
        arrayRet.protocol_header.method = 'LOKI-' + tagArray.type;
        if (tagArray.type == 'sip' && tagArray.method) arrayRet.protocol_header.method = 'LOKI-SIP-M-' + tagArray.method;
        else if (tagArray.type == 'sip' && tagArray.response) arrayRet.protocol_header.method = 'LOKI-SIP-R-' + tagArray.response;
        arrayRet.protocol_header.captureId = tagArray.node_id;
    }
    returnData.push(arrayRet);
};
return returnData
```
