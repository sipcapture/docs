# RTPPROXY + HEP

<img src="https://opengraph.githubassets.com/96bbc25bfaf9f9c4228742862d8d672f90113dc54d064cfbc593e4154b393a21/sippy/rtpproxy" width=600 />

### HEP Support
[RTPProxy](https://www.rtpproxy.org/) can natively send HEP RTCP Reports _(HEP type 5)_ for relayed RTP/RTCP streams

### Module Configuration Parameters:
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
