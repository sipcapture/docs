#SIPGREP
### Using Sipgrep as disposable HEP3 Agent

[Sipgrep](https://github.com/sipcapture/sipgrep) is a modern pcap-aware tool command line tool to capture, filter, display and help troubleshoot SIP signaling over IP networks, allowing the user to specify extended regular expressions matching against SIP headers and with nifty extra features.

[Sipgrep](https://github.com/sipcapture/sipgrep) is able to act as a quick on-demand HEP3 capture agent and forward packets to a collector very easily when a simple terminal check does not suffice. In the following example, Sipgrep is used to display the traffic of interest as well as log it to a remote location, useful for instance when troubleshooting issues on hosted platforms or disposable instances on the cloud.

<img src="http://i.imgur.com/lgJuGKz.png" width=500>

#### HEP3 Example:
Display dialogs and duplicate all traffic to HOMER sipcapture in HEPv3:
```
sipgrep -f 23333 -H udp:10.0.0.1:9061
```

#### General Examples:
```
# Find a dialog there From user contains '2323232'
sipgrep -f 2323232
# Find a dialog there To user contains '1111' and print dialog report
sipgrep -f 1111 -G
# Display only 603 replies without dialog match
sipgrep '^SIP/2.0 603' -m
# Display only OPTIONS and NOTIFY requests
sipgrep '^(OPTIONS|NOTIFY)'
# Display only SUBSCRIBE dialog
sipgrep 'CSeq:\s?\d* (SUBSCRIBE|PUBLISH|NOTIFY)' -M
# Collect all messages while pcap_dump smaller than 20kb
sipgrep -q 'filesize:20' -O sipgrep.pcap
# Kill friendly-scanner automatically
sipgrep -J
# Kill friendly-scanner with custom UAC name
sipgrep -j sipvicious
# Collect all Calls/Registrations dialogs during 120 seconds, print reports and exit:
sipgrep -g -G -q 'duration:120'
# Split pcap_dump to 20 KB files in sipgrep_INDEX_YYYYMMDDHHMM pcap
sipgrep -Q 'filesize:20' -O sipgrep.pcap
# Split pcap_dump in sipgrep_INDEX_YYYYMMDDHHMM.pcap each 120 seconds
sipgrep -Q 'duration:120' -O sipgrep.pcap
```
