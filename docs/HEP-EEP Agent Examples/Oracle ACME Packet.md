## ORACLE/ACME Packet & Homer
<!-- 
### IPFIX (new)

ORACLE /ACME PACKET Net-Net SBCs features a built in "Capture Agent" using a custom IPFIX template to export SIP messages and Statistics in realtime from the core. HORACLIFIX converts IPFIX to HEP for HOMER and HEPIC without requiring port mirroring and switches/probes/agents.

##### Setup
Install either HORACLIFIX (go) or HEPFIX (js) to translate IPFIX to HEP:

###### HORACLIFIX
```
go get http://github.com/negbie/horaclifix
go install http://github.com/negbie/horaclifix
```

Launch HORACLIFIX with command line parameters:
```
horaclifix -H 10.0.0.123:9060&
```

###### HEPFIX
```
git clone https://github.com/sipcapture/hepfix.js
cd hepfix.js
npm install
```

Configure HEPFIX with the HOMER IP:PORT details in ```config.js```
```
{
  ipfix_config: {
    debug: false,
    IPFIX_PORT: 4739
  },
  hep_config: {
    debug: false,
    HEP_SERVER: '10.0.0.123',
    HEP_PORT: 9060,
    HEP_ID: 2017,
    HEP_PASS: 'oracme'
  }
}
```
Once completed, start HEPFIX:
```
nodejs hepfix.js
```

------------------

### COMM-MONITOR

Next we will configure our ACME SBC comm-monitor to point at the HEPFIX IP and PORT
```
comm-monitor
    state           enabled
    qos-enable      enabled
    sbc-grp-id      0
    tls-profile
    monitor-collector
        address               192.168.122.1
        port                  4739
        network-interface     wancom0:0
```

<img src="https://camo.githubusercontent.com/266f73ec9fd3e9056f7ecdbeda4390a7ffffa8b4/687474703a2f2f692e696d6775722e636f6d2f45726b6a6939502e706e67">

--------------
-->

### IPIP (obsolete)

ORACLE / ACME ACLI provides a *"packet-trace"* command which can capture [RFC2003](https://tools.ietf.org/html/rfc2003) *(IP Encapsulation within IP)* SIP signaling on the Net-Net SBC and natively mirror it to your Homer SIP Capture server. 

To configure, we need to create a capture receiver:
```
capture-receiver
        state               enabled
        ip-address          192.168.0.1
        network-interface   m00:0
        last-modified-by    engineer@192.168.30.1
        last-modified-date  2011-10-10 00:00:01
```

or alternatively:

```
packet-trace-config < state | address | network-interface | select| no | show | done | exit>
```

The *"packet-trace"* command on the Net-Net SBC can now duplicate all packets sent to and from the endpoint identified by the IP address on the specified Net-Net SBC network interface:
```
#packet-trace start Access:0 192.168.30.5 5060 5060
```

If you access and network side present different *Call-ID* header breaking call-flow correlation, you can use an *HMR* to inject a X-CID: header containing the Call-ID from A-leg into INVITE in your B-leg;

##### *Fictional HMR Example:*

```
sip-manipulation
        name                           xCID
        description                    Create_XCID
        split-headers
        join-headers
        header-rule
                name                           XCIDstore
                header-name                    Call-ID
                action                         store
                comparison-type                pattern-rule
                msg-type                       out-of-dialog
                methods                        INVITE
                match-value                    .*
                new-value
        header-rule
                name                   	       DeleteXCID
                header-name            	       X-CID
                action                 	       delete
                comparison-type        	       pattern-rule
                msg-type               	       request
                methods                	       INVITE
                match-value            	       .*
                new-value
        header-rule
                name                           addXCID
                header-name                    X-CID
                action                         add
                comparison-type                boolean
                msg-type                       out-of-dialog
                methods                        INVITE
                match-value                    !$XCIDstore
                new-value                      $XCIDstore.$0
```


---------------------

## CDR QoS Correlation
The following table provides a general mapping convention for ACME/ORACLE SBC CDRs

###Leg 1 Voice Quality Report
```
| QoS Metric        | CDR Parameter|
| ------------- |:------------:| 
|"POST_DIAL_DELAY"|Acme-Post-Dial-Delay (PDD MAP)|
|"DISCONNECT_INIT"|Acme-Disconnect-Initiator (INIT MAP)|
|"DISCONNECT_CAUSE"|Acme-Disconnect-Cause (CAUSE MAP)|
|"LAST_STATUS"|Acme-Session-Disposition (STATUS MAP)|
|"FIRST_SWITCHED" | MAS-Setup-Time (Epoch conversion)|
|"IPV4_SRC_ADDR" | Acme-Flow-Out-Dst-Addr_FS1_R|
|"L4_SRC_PORT" | Acme-Flow-Out-Dst-Port_FS1_R|
|"IPV4_DST_ADDR" | Acme-Flow-Out-Src-Addr_FS1_R|
|"L4_DST_PORT" | Acme-Flow-Out-Src-Port_FS1_R|
|"RTP_IN_JITTER": | Acme-Calling-RTCP-MaxJitter_FS1|
|"RTP_OUT_JITTER" | Acme-Calling-RTP-MaxJitter_FS1|
|"RTP_IN_PKT_LOST" | Acme-Calling-RTCP-Packets-Lost_FS1|
|"RTP_OUT_PKT_LOST" | Acme-Calling-RTP-Packets-Lost_FS1|
|"OUT_BYTES" | Acme-Calling-Octets_FS1|
|"OUT_PKTS" | Acme-Calling-Packets_FS1|
|"IN_BYTES" | Acme-Called-Octets_FS1|
|"IN_PKTS" | Acme-Called-Packets_FS1|
|"RTP_IN_TRANSIT" | Acme-Calling-RTCP-MaxLatency_FS1|
|"RTP_IN_PAYLOAD_TYPE" | Acme-FlowType_FS1_R (CODEC MAP)|
|"RTP_OUT_PAYLOAD_TYPE" | Acme-FlowType_FS1_F (CODEC MAP)|
|"RTP_SIP_CALL_ID" | Acct-Session-Id|
|"RTP_MOS" | Acme-Calling-MOS|
|"RTP_R_FACTOR" | Acme-Calling-R-Factor|
```

###Leg 2 Voice Quality Report
```
| QoS Metric        | CDR Parameter|
| ------------- |:------------:| 
|"FIRST_SWITCHED" | MAS-Setup-Time (Needs Epoch conversion)|
|"IPV4_SRC_ADDR" | Acme-Flow-Out-Src-Addr_FS1_F|
|"L4_SRC_PORT" | Acme-Flow-Out-Src-Port_FS1_F|
|"IPV4_DST_ADDR" | Acme-Flow-Out-Dst-Addr_FS1_F|
|"L4_DST_PORT" | Acme-Flow-Out-Dst-Port_FS1_F|
|"RTP_IN_JITTER": | Acme-Called-RTP-MaxJitter_FS1|
|"RTP_OUT_JITTER" | Acme-Called-RTCP-MaxJitter_FS1|
|"RTP_IN_PKT_LOST" | Acme-Called-RTP-Packets-Lost_FS1|
|"RTP_OUT_PKT_LOST" | Acme-Called-RTCP-Packets-Lost_FS1|
|"OUT_BYTES" | Acme-Calling-Octets_FS1|
|"OUT_PKTS" | Acme-Calling-Packets_FS1|
|"IN_BYTES" | Acme-Called-Octets_FS1|
|"IN_PKTS" | Acme-Called-Packets_FS1|
|"RTP_OUT_TRANSIT" | Acme-Called-RTCP-MaxLatency|
|"RTP_IN_PAYLOAD_TYPE" | Acme-FlowType_FS1_F (CODEC MAP)|
|"RTP_OUT_PAYLOAD_TYPE" | Acme-FlowType_FS1_R (CODEC MAP)|
|"RTP_SIP_CALL_ID" | Acct-Session-Id|
|"RTP_MOS" | Acme-Called-MOS|
|"RTP_R_FACTOR" | Acme-Called-R-Factor|
```
