# Native __HEP__ Agents

Native HEP Agents are available in [OpenSIPS](https://github.com/sipcapture/homer/wiki/Examples%3A-OpenSIPS), [Kamailio](https://github.com/sipcapture/homer/wiki/Examples%3A-Kamailio), [Asterisk](https://github.com/sipcapture/homer/wiki/Examples%3A-Asterisk), [Freeswitch](https://github.com/sipcapture/homer/wiki/Examples%3A-FreeSwitch), [RTP:Engine](https://github.com/sipcapture/homer/wiki/Examples%3A-RTPEngine) and many more. Consult the Wiki to get specific examples for your platform.

<!-- tabs:start -->

## ** OpenSIPS **

![](http://i.imgur.com/MoOOI23.png)

### HEP in OpenSIPS _(proto_hep)_
With version 2.2 a new protocol emerged integrating almost everything related to HEP. The module is called proto_hep and implements HEP messages encapsulation both on the client and the server side. The module offers support for older protocols such as HEPv1 and HEPv2 that were already impelemented by sipcapture and siptrace modules and also offers support for the new HEPv3. Versions 1 and 2 have only UDP support while version 3 offers support for TCP and UDP.

What is more, since version 2.3 all the information concerning the destination of the HEP packets have to be defined in this module. If your OpenSIPS instance also acts as a HEP server one can define HEP listeners to handle the HEP traffic.

##### HEP Tracing Overview
Until version 2.3 OpenSIPS was able to trace only the SIP packets that were processed in any way by the SIP proxy. With version 2.3 came the major improvement that allows tracing other nonSip events such as:

* sip-context events _(they depend on Sip events to be triggered)_ - such traced events are rest queries and xlog messages;
* sip-triggering events _(such events will most probably trigger future Sip events)_ - connection events tracing for TCP, WS, WSS and TLS modules; here we will trigger HEP packets when a connection is opened or closed;
* async events(they are - almost always - totally unrelated to Sip events) - mi commands _(all mi modules are supported)_;

All the new events that can be traced will help you have a much better picture over how your sip proxy is behaving. The events that are depending on Sip can be set using sip_trace function from siptrace module. All the other events will have a configurable trace destination that will tell where the messages will be sent. All the events will need proto_hep module which will provide the client interface for sending HEP messages.

#### HEP Capturing Overview _(sipcapture)_
Sipcapture is a module capturing HEP packets and not only, as the name suggests. Version 2.2 came with major additions to sipcapture module concerning handling HEP messages such as all the functions allowing you to change the received HEP message (hep_set, hep_get and hep_del) or hep_relay which transforms your capturing node into a HEP proxy and hep_route parameter which allows choosing the path your Sip messages should take.

##### Tracing
###### Prerequisites
As discussed before, first thing before using any HEP tracing capabilities, `proto_hep` module will have to be included.
```
   loadmodule "proto_hep.so"
```

VERY IMPORTANT: in order to be able to send or receive any packets a HEP listener will have to be defined:
```
   #TCP Hep listener
   listen = hep_tcp:IP:HEP_TCP_PORT 		# change the listening IP
   #UDP Hep listener
   listen = hep_udp:IP:HEP_UDP_PORT 		# change the listening IP
```

If version 2.3 or newer is used a hep id will also have to be defined for identifying the receiving endpoint for the HEP packets
```
   modparam("proto_hep", "hep_id",
      "[hep_dst] RECEIVING_IP:RECEIVING_TRANSPORT; transport=tcp; version=3")
```

Also new in version 2.3 are two parameters, homer5_on and homer5_delim. By default newer versions will encode the payload of the message as JSON since this is the protocol that the newest version of HOMER will use. This may cause failures on the capturing node so using these two parameters you can choose that all values in the JSON to be set in some sort of CSV format with a delimiter you can choose in order to make your job easier on the capturing node.
```
    ### activate homer5 CSV like format for the payload
    modparam("proto_hep", "homer5_on", 1)
    ### set the delimiter for the payload 
    modparam("proto_hep", "homer5_delim", "##")
```

###### SIP Tracing
Having done all the setups that are required to be able to send HEP packets now we can configure siptrace module. First of all we need to include the module and define a siptrace_id using the hep id previously defined in proto_hep module.
```
   loadmodule "siptrace.so"
   modparam("siptrace", "trace_id", "[hep_tid]hep:hep_dst")
   ### in 2.2 the full specification of the uri have to be done here
   # modparam("siptrace", "trace_id", "[hep_tid] uri=hep:DST_IP:HEP_DST_PORT; version=3;transport=tcp") 
```   
   
What is left to do in the script is to call sip_trace function whenever tracing is desired. It's your decision to take where and when you want to enable tracing. As an example let's suppose we want to trace the dialogs for all the registered users:
```
    ...
    route(AUTHENTICATE);
    ### if here the user is valid and we will trace this dialog 
    ### the tracing in this scenario will have to be activated only on the initial invite
    ###    and all the rest of messages belonging to this dialog shall be traced
    if ( !has_totag() && is_method("INVITE") )
        sip_trace("hep_tid","d");
    ### also working only in 2.3
    ### sip is traced by default so there's no need to force it
    # sip_trace("hep_tid","d", "sip");
```

For further details and examples please consult the [OpenSIPS Documentation](https://www.opensips.org/Documentation/Tutorials-Tracing)


## ** Kamailio **

![](http://kb.asipto.com/images/kamailio.jpg)

### KAMAILIO HEP Tracing

SIP capture functionalities are built into core kamailio. We have to only load required module, initialize it with the appropriate parameters and modify routing logic to use it. To keep the changes flexible and clean, this excample uses directives which allow us to simply switch on/off the additional functionality:

##### kamailio.cfg
First, define WITH_HOMER directive at the head of your script:
```
#!define WITH_HOMER
```

Next, move to the loadparam section of the script, and add a condition for siptrace:
```
#!ifdef WITH_HOMER
loadmodule "siptrace.so"
#!endif
```

_NOTE: siptrace module MUST be loaded after loading mysql and tm modules!_

Next, configure the basic parameters for the module:
```
#!ifdef WITH_HOMER
# check IP and port of your capture node
modparam("siptrace", "duplicate_uri", "sip:10.0.0.1:9060")
modparam("siptrace", "hep_mode_on", 1)
modparam("siptrace", "trace_to_database", 0)
modparam("siptrace", "trace_flag", 22)
modparam("siptrace", "trace_on", 1)
#!endif
```


Finally, use SIPTRACE in your ```route {}``` logic where needed:

```
#!ifdef WITH_HOMER
        #start duplicate the SIP message now
        sip_trace();
        setflag(22);
#!endif
```

### KAMAILIO 5.x Trace Node

##### kamailio.cfg
First, define WITH_HOMER directive at the head of your script:
```
#!define WITH_HOMER
```

Next, move to the loadparam section of the script, and add a condition for siptrace:
```
#!ifdef WITH_HOMER
loadmodule "siptrace.so"
#!endif
```

_NOTE: siptrace module MUST be loaded after loading mysql and tm modules!_

Next, configure the basic parameters for the module:
```
#!ifdef WITH_HOMER
# check IP and port of your capture node
modparam("siptrace", "duplicate_uri", "sip:10.0.0.1:9060")
# Send from an IP
modparam("siptrace", "send_sock_addr", "sip:10.2.0.2:5000")
modparam("siptrace", "hep_mode_on", 1)
modparam("siptrace", "trace_to_database", 0)
modparam("siptrace", "trace_flag", 22)
modparam("siptrace", "trace_on", 1)
#!endif
```

Finally, use 'sip_trace' in your route {} logic where needed:


```
#!ifdef WITH_HOMER
        setflag(22);

        #start duplication mode: m or M for message; t or T for transaction; d or D for dialog
        sip_trace_mode("t");

        #start duplicate the SIP message now 
        sip_trace();

        # Or you can use new syntax, if you want to have multipe copy of your data
        sip_trace("sip:10.0.0.2:9060");

        # send to 10.0.0.3 and assign a callid as correlation param
        sip_trace("sip:10.0.0.3:9060", "$ci-abc");

        # or by dialog: trace current dialog; needs to be done on initial INVITE and dialog has to be loaded
        sip_trace("sip:10.0.0.4:9060", "$ci-abc", "d");
          
        #Send HEPLog:
        hlog("$hdr(P-MyID)", "Another one with a custom correlation ID");
  
#!endif
```



## ** Freeswitch **

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


## ** Asterisk **

![](http://www.asterisk.org/sites/asterisk/themes/asterisk/logo.png ':size=100')

Asterisk 12+ ships with HEP encapsulation support (res_hep) and is able to natively mirror its packets to a HEP/EEP Collector such as HOMER. When using ```chan_pjsip``` enabling HEP features is as simple as configuring ```/etc/asterisk/hep.conf```

### Requirements

* loading ```res_hep_pjsip.so``` module

### Example Configuration:
<pre>
/etc/asterisk/hep.conf

;
; res_hep Module configuration for Asterisk
;

; All settings are currently set in the general section.
[general]
<b>enabled = yes </b>
; Enable/disable forwarding of packets to a
; HEP server. Default is "yes".
<b>capture_address = 10.0.0.1:9060 </b>
; The address of the HEP capture server.
<b>capture_password = foo </b>
; If specified, the authorization passsword
; for the HEP server. If not specified, no
; authorization password will be sent.
<b>capture_id = 1234 </b>
; A unique integer identifier for this
; server. This ID will be embedded sent
; with each packet from this server.
</pre>

Once configured and enabled, the module will begin forwarding all handled SIP packets to HOMER for handling.


----------

### RTCP statistics

Asterisk 12+ ships with _res_hep_rtcp_. The module subscribes to Stasis and receives RTCP information back from the message bus, which it encodes into HEPv3 packets and sends to the res_hep module for transmission. Using this module, someone with a Homer server can get live call quality monitoring for all channels in their Asterisk 12+ systems.

#### Enable
To enable the functionality, just load the module alongside the [[res_hep|Examples:-Asterisk]] module

------------

The module supports sender (SR) and receiver (RR) reports:

* rtcp-sender: Audio is streamed from the first instance and echo'd back from the second to the first. This results in RTCP information from both instances being sent to a HEP server, where both sides are 'senders' and thus generate/receive SR packets.
* rtcp-receiver: Audio is streamed from the first instance and absorbed by the second. This results in RTCP information from both instances being sent to a HEP server, where the first transmits SR packets and received RR packets and the second transmits RR packets and received SR packets.


-------------

### PJSIP X-CID Correlation for BLEG
```
PJSIP_HEADER(add,X-CID)=$SIPCALLID
```

### CDR Correlation Example

##### /etc/asterisk/extensions.conf
```
exten => s,1,Set(CDR(callid)=${SIPCALLID})
exten => s,2,Set(CDR(rtcpinfo)=${RTPAUDIOQOS})
```

##### /etc/asterisk/cdr_custom.conf
```
[mappings]
Master.csv => ${CSV_QUOTE(${CDR(clid)})},${CSV_QUOTE(${CDR(src)})},${CSV_QUOTE(${CDR(dst)})},${CSV_QUOTE(${CDR(dcontext)})},${CSV_QUOTE(${CDR(channel)})},${CSV_QUOTE(${CDR(dstchannel)})},${CSV_QUOTE(${CDR(lastapp)})},${CSV_QUOTE(${CDR(lastdata)})},${CSV_QUOTE(${CDR(start)})},${CSV_QUOTE(${CDR(answer)})},${CSV_QUOTE(${CDR(end)})},${CSV_QUOTE(${CDR(duration)})},${CSV_QUOTE(${CDR(billsec)})},${CSV_QUOTE(${CDR(disposition)})},${CSV_QUOTE(${CDR(amaflags)})},${CSV_QUOTE(${CDR(accountcode)})},${CSV_QUOTE(${CDR(uniqueid)})},${CSV_QUOTE(${CDR(userfield)})},${CDR(sequence)},${CDR(callid)},${CDR(rtcpinfo)}
```

## ** RTP:Engine **

![](https://www.sipwise.com/wp-content/themes/sipwise/assets/images/logo.svg) 

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



### ** RTPProxy **

<img src="https://opengraph.githubassets.com/96bbc25bfaf9f9c4228742862d8d672f90113dc54d064cfbc593e4154b393a21/sippy/rtpproxy" width=300 />

#### HEP Support
[RTPProxy](https://www.rtpproxy.org/) can natively send HEP RTCP Reports _(HEP type 5)_ for relayed RTP/RTCP streams

#### Module Configuration Parameters:
Create _(or extend)_ the optional configuration file containing the required settings:
```
modules {
   rtpp_acct_rtcp_hep {
       load = ../modules/acct_rtcp_hep/.libs/rtpp_acct_rtcp_hep.so
       capt_host  = HEP_SERVER_IP
       capt_port  = 9060
       capt_ptype = udp
       capt_id = 101
   }
}
```

Include the saved configuration in your RTPProxy arguments using the config parameter:
```
RTPPROXY_ARGS="--config path/to/rtpp_acct_rtcp_hep.conf"
```

Further examples can be found in the [RTPProxy test coverage](https://github.com/sippy/rtpproxy/blob/master/tests/acct_rtcp_hep/basic.conf)

<!-- tabs:end -->

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>
