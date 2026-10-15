
![](http://kb.asipto.com/images/kamailio.jpg)
<!--
## Kamailio as HOMER Capture server

Homer's sipcapture module allows Kamailio to operate as a robust and scalable SIP sampling/capture server 
with native support for HEPv1/v2, IPIP Encapsulation protocols and switch mirroring/monitoring port 
traffic. Kamailio can be configured either as Capture Agent (siptrace module) sampling and forwarding packets, or as Capture Node (sipcapture module) collecting, indexing and storing SIP packets as received  
from the available Capture Agents (HEP), SBCs (IPIP) or directly from the ethernet wire.

Capture Agents can be distributed in a modular fashion, allowing support for any network topology.
In addition to the integrated sampling and capturing functions in Kamailio, a stand-alone capture 
agent (captagent) is provided enabling HEP encapsulation for unsupported systems and soft-switches.
OpenSIPS/SER and FreeSWITCH users already enjoy integrated HEP mirroring functionality.

![](http://homer.googlecode.com/files/HOMER-Kamailio.png)

### Capture node config:

```
!KAMAILIO
#
####### Global Parameters #########
debug=1
log_stderror=no
memdbg=5
memlog=5
log_facility=LOG_LOCAL0
fork=yes
children=5
disable_tcp=yes

/* IP and port for HEP capturing) */
listen=udp:10.0.0.1:9060

/* enable it only in mirroring scenario, not for HEP! */
/* #!define SIPCAPTURE_MIRRORING_PORT */

mpath="/usr/local/lib64/kamailio/modules_k/:/usr/local/lib64/kamailio/modules/"

loadmodule "pv.so"
loadmodule "db_mysql.so"
loadmodule "sipcapture.so"

# ----- mi_fifo params -----

####### Routing Logic ########
modparam("sipcapture", "db_url", "mysql://homer:password@localhost/homer_data")
modparam("sipcapture", "capture_on", 1)
modparam("sipcapture", "table_name", "sip_capture")
modparam("sipcapture", "hep_capture_on", 1)
modparam("siptrace", "hep_capture_id", 301)
modparam("siptrace", "hep_version", 2)

#!ifdef SIPCAPTURE_MIRRORING_PORT
/* IP to listen. Port/Portrange apply only on mirroring port capturing */
modparam("sipcapture", "raw_socket_listen", "192.168.254.1:5060-5080")
/* Name of interface to bind on raw socket */
modparam("sipcapture", "raw_interface", "eth1")
/* activate monitoring/mirroring port capturing */
modparam("sipcapture", "raw_moni_capture_on", 1)
/* children for raw socket */
modparam("sipcapture", "raw_sock_children", 4)

/* Linux only */
/* Promiscious mode RAW socket. Mirroring port. */
modparam("sipcapture", "promiscious_on", 1)
/* activate BPF */
modparam("sipcapture", "raw_moni_bpf_on", 1)

#endif

/* insert delayed */
#modparam("sipcapture", "db_insert_mode", 1)


# Main SIP request routing logic
# - processing of any incoming SIP request starts with this route
route {

        #For example, you can capture only needed methods...
        if (!(method =~ "^(NOTIFY|SUBSCRIBE|OPTIONS)"))) {
                sip_capture();
        }
        drop;
}

onreply_route {

        #And replies of request methods
        if(!($rm =~ "^(NOTIFY|SUBSCRIBE|OPTIONS)")) {
                sip_capture();
        }
        drop;
}

```

### Trace node config:

```
#!KAMAILIO

debug=1
log_stderror=no

memdbg=5
memlog=5

log_facility=LOG_LOCAL0

fork=yes
children=4

disable_tcp=yes

listen=udp:192.168.0.1:5060

/* port to listen to
 * - can be specified more than once if needed to listen on many ports */
port=5060

####### Modules Section ########

mpath="/usr/local/lib64/kamailio/modules_k/:/usr/local/lib64/kamailio/modules/"

loadmodule "mi_fifo.so"
loadmodule "kex.so"
loadmodule "tm.so"
loadmodule "sl.so"
loadmodule "rr.so"
loadmodule "pv.so"
loadmodule "maxfwd.so"
loadmodule "xlog.so"
loadmodule "textops.so"
loadmodule "siputils.so"
loadmodule "siptrace.so"


modparam("mi_fifo", "fifo_name", "/tmp/kamailio_fifo")
modparam("tm", "failure_reply_mode", 3)
modparam("tm", "fr_timer", 30000)
modparam("tm", "fr_inv_timer", 120000)
modparam("rr", "enable_full_lr", 1)
modparam("rr", "append_fromtag", 0)

#Siptrace
modparam("siptrace", "duplicate_uri", "sip:10.0.0.1:9060")
modparam("siptrace", "hep_mode_on", 1)
modparam("siptrace", "trace_to_database", 0)
modparam("siptrace", "trace_flag", 22)
modparam("siptrace", "trace_on", 1)
modparam("siptrace", "hep_version", 3)

####### Routing Logic ########

# Main SIP request routing logic
# - processing of any incoming SIP request starts with this route
route {

        ....
        #start duplicate the SIP message now
        sip_trace();

        setflag(22);

        ....
        route(RELAY);
}

route[RELAY] {

        if (!t_relay()) {
                sl_reply_error();
        }
        exit;
}

```

-->


### KAMAILIO 4.x Trace Node

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
