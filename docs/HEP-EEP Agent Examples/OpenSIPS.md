![](http://i.imgur.com/MoOOI23.png)

## OpenSIPS 2.2 as HEP/EEP Switch
Please see the [dedicated wiki page](https://github.com/sipcapture/homer/wiki/Examples%3A-opensips-hepswitch)
## OpenSIPS as Homer Capture server

Homer's sipcapture module allows OpenSIPS to operate as a robust and scalable SIP sampling/capture server 
with native support for HEPv1/v2, IPIP Encapsulation protocols and switch mirroring/monitoring port 
traffic. OpenSIPS can be configured either as Capture Agent (siptrace module) sampling and forwarding packets, or as Capture Node (sipcapture module) collecting, indexing and storing SIP packets as received  
from the available Capture Agents (HEP), SBCs (IPIP) or directly from the ethernet wire.

Capture Agents can be distributed in a modular fashion, allowing support for any network topology.
In addition to the integrated sampling and capturing functions in OpenSIPS, a stand-alone capture 
agent (captagent) is provided enabling HEP encapsulation for unsupported systems and soft-switches.
OpenSER/Kamailio and FreeSWITCH users already enjoy integrated HEP mirroring functionality.

![](http://homer.googlecode.com/files/HOMER-OpenSIPS.png)

### OpenSIPs - config for capture node.

```
####### Global Parameters #########

debug=3
log_stderror=no
log_facility=LOG_LOCAL0

fork=yes
children=5

disable_tcp=yes
db_default_url="mysql://opensips:opensipsrw@localhost/opensips"
port=9060

/* uncomment and configure the following line if you want opensips to
   bind on a specific interface/port/proto (default bind on all available) */
listen=udp:10.0.0.1:9060

####### Modules Section ########
#set module path
mpath="/usr/local/lib64/opensips/modules/"

loadmodule "db_mysql.so"
loadmodule "sipcapture.so"

####### Routing Logic ########
modparam("sipcapture", "db_url", "mysql://homer:password@localhost/homer_db")
modparam("sipcapture", "capture_on", 1)
modparam("sipcapture", "table_name", "sip_capture")
/* activate HEP capturing */
modparam("sipcapture", "hep_capture_on", 1)

/* configuration for Mirroring PORT */
modparam("sipcapture", "raw_socket_listen", "10.0.130.41:5060-6000")
modparam("sipcapture", "raw_interface", "eth1")
/* activate monitoring port capturing */
modparam("sipcapture", "raw_moni_capture_on", 1)
modparam("sipcapture", "raw_sock_children", 4)
/* Promiscious mode */
modparam("sipcapture", "promiscious_on", 1)

####### Routing Logic ########


# main request routing logic

# Main SIP request routing logic
# - processing of any incoming SIP request starts with this route
route {
        #For example, you can capture only needed methods...
        #if (!(method =~ "^(OPTIONS|NOTIFY|SUBSCRIBE)$"))) {
                sip_capture();
        #}
        drop;
}

onreply_route {

        #And only needed reply or needed requests method
        #if(status =~ "^(1[0-9][0-9]|[3[0-9][0-9]|4[0-9]|[56][0-9][0-9])") {
        #if(!($rm =~ "^(NOTIFY|SUBSCRIBE|OPTIONS|)$")) {
                sip_capture();
        #}
        drop;
}



```

### OpenSIPs - config for trace node.

```

####### Global Parameters #########
debug=3
log_stderror=no
log_facility=LOG_LOCAL0
fork=yes
children=4
disable_tcp=yes
db_default_url="mysql://opensips:opensipsrw@localhost/opensips"

port=5060

listen=udp:192.168.0.1:5060


####### Modules Section ########

#set module path
mpath="/usr/local/lib64/opensips/modules/"

loadmodule "signaling.so"
loadmodule "sl.so"
loadmodule "tm.so"
loadmodule "rr.so"
loadmodule "maxfwd.so"
loadmodule "textops.so"
loadmodule "mi_fifo.so"
loadmodule "siptrace.so"


# ----------------- setting module-specific parameters ---------------

# ----- mi_fifo params -----
modparam("mi_fifo", "fifo_name", "/tmp/opensips_fifo")
modparam("rr", "append_fromtag", 0)
modparam("siptrace", "duplicate_uri", "sip:10.0.0.1:9060")
modparam("siptrace", "duplicate_with_hep", 1)
modparam("siptrace", "trace_to_database", 0)
modparam("siptrace", "trace_flag", 22)
modparam("siptrace", "trace_on", 1)
#HEPv2 == timestamp will be included to HEP header
modparam("siptrace", "hep_version", 2)

 Main SIP request routing logic
# - processing of any incoming SIP request starts with this route
route {

        ......

        setflag(22);

        #Duplicate this sip message to Homer capture node
        sip_trace(); 

        .....
        route(1);
}

route[1] {

        if (!t_relay()) {
                sl_reply_error();
        };
        exit;
}

```

----



### SIPTRACE on OpenSIPS 1.7.x:

To enable HEP support in siptrace/opensips 1.7.x do the following:

1. Download OpenSIPS SVN head

2. Copy modules/siptrace and modules/sipcapture to the 1.7.1 source tree

3. Modify modules/siptrace/siptrace.c as follows:
```
        -     if (dlgb.create_dlg(msg,0)<1) {
        +     if (dlgb.create_dlg(msg)<1) {
```
4. Recompile the siptrace module:
```
        # make modules=modules/siptrace modules
```

_NOTE: Module SIPCAPTURE is only used to compile module SIPTRACE and CANNOT be used on OpenSIPS 1.7.x (for Homer Capture always use TRUNK)_
