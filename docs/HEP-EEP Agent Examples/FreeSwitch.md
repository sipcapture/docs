![FreeSwitch](http://i.imgur.com/gylbBjz.png)
## FreeSWITCH Capture Agent

Freeswitch ships with an integrated HEP Capture Agent designed to work with HOMER

**FreeSwitch HEP3/EEP support is available in 1.6.8+ **

### Global Configuration
To enable HEP capturing, open sofia.conf.xml and set capture-server param

```
<param name="capture-server" value="udp:192.168.0.1:9060"/>
```

Freeswitch >= 1.7 has support for HEPv2 and HEPv3. The new syntax is:

```
<param name="capture-server" value="udp:192.168.0.1:9060;hep=3;capture_id=100"/>
```
open internal.xml and external.xml and change sip-capture param to "yes". Also please do it on each profile in your sip_profiles/internal and sip_profiles/external (/etc/freeswitch/sip_profiles)

```
<param name="sip-capture" value="yes"/>
```
*note: the ip address and port must be same as the listen param in your kamailio.cfg or in heplify-server *

----

To enable/disable the HEP agent on demand, you can use CLI commands:
```
freeswitch@fsnode04> sofia global capture on
 
+OK Global capture on
freeswitch@fsnode04> sofia global capture off
 
+OK Global capture off
```

### Profile Configuration

You can choose to activate HEP capturing only for a specific profile:
```
freeswitch@fsnode04> sofia profile internal capture on
 
Enabled sip capturing on internal

freeswitch@fsnode04> sofia profile internal capture off
 
Disabled sip capturing on internal
``` 

### B2BUA Correlation
To correlate B2BUA legs set the following before bridging the second leg:
```
      <action application="set" data="sip_h_X-cid=${sip_call_id}"/>
```

Next, configure `heplify-server` to extract correlation from the newly created `X-cid` header _(TODO: link)_

### ESL Integration (beta)

[hepipe.js](https://github.com/sipcapture/hepipe.js/blob/master/esl) provides **experimental** support for FreeSWITCH ESL integration for call quality reports feeding to HOMER 5, effectively providing external HEP3/EEP features with correlation support.

#### Events

| ESL Event  | Hep Mode | HEP Type  |
|:--|:--|:--|
| CHANNEL_CREATE | LOG | 100 |
| CHANNEL_ANSWER | LOG | 100 |
| CHANNEL_DESTROY | LOG | 100 | 
| CUSTOM | LOG | 100 | 
| RECV_RTCP_MESSAGE | RTCP | 5 | 
| CHANNEL_DESTROY | CUSTOM QoS | 99 |

For full instructions and details please checkout [hepipe.js](https://github.com/sipcapture/hepipe.js/blob/master/esl)

If you test or extend this feature please share your feedback!
