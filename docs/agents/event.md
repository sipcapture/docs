# Event __HEP__ Agents

HOMER can collect, index and correlate non-packet events such as Logs, RTC stats, CDRs, and more using [HEP](https://github.com/sipcapture/hep) supported by a variety of tools such as [paStash](https://github.com/sipcapture/pastash/wiki) and [Telegraf](https://github.com/influxdata/telegraf/pull/6167)

<!-- tabs:start -->

## ** HFP **

HPF provides reliable TCP HEP delivery to any HEP remote server behind unreliable networks.

### Usage
Download a [release](https://github.com/sipcapture/HFP/releases) for your system to start using HFP
```
./hfp -l :9060 -r (HEP TCP server we want to reliably proxy HEP)
```

#### Options
```
  -l string
    	Local HEP listening address (default ":9060")
  -r string
    	Remote HEP address (default "192.168.2.2:9060")
  -ipf string
    	IP filter address from HEP SRC or DST chunks. Option can use multiple IP as comma sepeated values. Default is no filter without processing HEP acting as high performance HEP proxy
  -ipfa string
    	IP filter Action. Options are pass or reject (default "pass")
  -d string
    	Debug options are off or on (default "off")
  -prom string
    	Prometheus metrics port (default "8090")
  -v  
      Prints current HFP version
```

> HFP only works with TCP/TLS and does not support UDP/HEP streams

## ** paStash **

[paStash](https://github.com/sipcapture/paStash/wiki) can be used to ingest non-SIP events into HOMER from a variety of sources.

Check out the [paStash](https://github.com/sipcapture/paStash) repository for recipes and install options.

<!-- tabs:end -->

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>
