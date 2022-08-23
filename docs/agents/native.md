### **Native Agents**
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

## ** Asterisk **

## ** RTP:Engine **

## ** RTPProxy **

<!-- tabs:end -->

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>
