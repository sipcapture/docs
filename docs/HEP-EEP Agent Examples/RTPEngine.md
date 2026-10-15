![](https://www.sipwise.com/wp-content/themes/sipwise/assets/images/logo.svg) 

RTPEngine
=======================

The [Sipwise](http://www.sipwise.com/) NGCP rtpengine is a proxy for RTP traffic and other UDP based
media traffic. It's meant to be used with the [Kamailio SIP proxy](http://www.kamailio.org/) and [OpenSIPS SIP proxy](http://www.opensips.org/) and forms a drop-in replacement for any of the other available RTP and media proxies.

RTPEngine mr4.4.1+ supports [HEP3](http://hep.sipcapture.org) Encapsulation and can mirror RTCP packets relayed between streams to **HOMER** complete with SIP correlation Call-IDs from the respective signaling session. 

#### NGCP Users
Users of the NGCP platform should enable HOMER support directly in their ```config.yml``` replacing the destination parameter with the correct HOMER HEP socket and applying with ```ngcpcfg apply```
```
...
homer_rtcp_stats:
    destination: 10.20.30.40:9070
    enabled: yes
    id: '2001'
    protocol: udp
...
```


##### Parameters
```
 --homer=IP46:PORT           Address of Homer server for RTCP stats
 --homer-protocol=udp|tcp    Transport protocol for Homer (must be defined)
 --homer-id=INT              'Capture ID' to use within the HEP protocol
```

### Setup
Configure RTPEngine to send RTCP reports to Homer using the same HEP settings you used in your Kamailio/OpenSIPS/Captagent configuration to achieve SIP session correlation:

![](http://i.imgur.com/cWB7eWh.png)

##### Example HEP Settings (command line)
```
 --homer=10.0.0.20:9060 
 --homer-protocol=udp  
 --homer-id=999 
```

```
/usr/sbin/rtpengine --interface=10.0.0.1 --listen-tcp=25060 \
--listen-udp=12222 --listen-ng=22222 --listen-cli=9900 --timeout=60 \
--silent-timeout=3600 --pidfile=/var/run/ngcp-rtpengine-daemon.pid \
--table=0 --log-level=6 --log-facility=daemon --homer=10.0.0.20:9060 \
--homer-id=999 --homer-protocol=udp
```

##### Example HEP Settings (rtpengine.default)
```
HOMER=10.0.0.20:9060
HOMER_PROTOCOL=udp
HOMER_ID=2099
```

##### Example RTCP Report
```
{"sender_information":
{"ntp_timestamp_sec":3667834977,"ntp_timestamp_usec":2355549043,
"octets":82240,"rtp_timestamp":84679,"packets":514},
"ssrc":3724882677,"type":200,"report_count":1,"report_blocks":
[{"source_ssrc":2062957521,"highest_seq_no":4663,"fraction_lost":0,
"ia_jitter":13,"packets_lost":0,"lsr":3093100159,"dlsr":296222}],
"sdes_ssrc":3724882677,"sdes_report_count":1,"sdes_information": [] }
```

##### Example QoS Report

![](https://camo.githubusercontent.com/6394f82d5a0511085f4f6980e2b31ca6fe334e38/687474703a2f2f692e696d6775722e636f6d2f38784b62513837672e706e67)


<br>
