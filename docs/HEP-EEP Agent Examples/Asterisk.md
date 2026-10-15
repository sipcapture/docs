# ![](http://www.asterisk.org/sites/asterisk/themes/asterisk/logo.png)


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

# RTCP statistics

Asterisk 12+ ships with _res_hep_rtcp_. The module subscribes to Stasis and receives RTCP information back from the message bus, which it encodes into HEPv3 packets and sends to the res_hep module for transmission. Using this module, someone with a Homer server can get live call quality monitoring for all channels in their Asterisk 12+ systems.

## Enable
To enable the functionality, just load the module alongside the [[res_hep|Examples:-Asterisk]] module

------------

The module supports sender (SR) and receiver (RR) reports:

* rtcp-sender: Audio is streamed from the first instance and echo'd back from the second to the first. This results in RTCP information from both instances being sent to a HEP server, where both sides are 'senders' and thus generate/receive SR packets.
* rtcp-receiver: Audio is streamed from the first instance and absorbed by the second. This results in RTCP information from both instances being sent to a HEP server, where the first transmits SR packets and received RR packets and the second transmits RR packets and received SR packets.


-------------

# PJSIP X-CID Correlation for BLEG
```
PJSIP_HEADER(add,X-CID)=$SIPCALLID
```

# CDR Correlation Example

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
