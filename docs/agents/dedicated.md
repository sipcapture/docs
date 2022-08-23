### **Dedicated Agents**

HOMER features dedicated _HEP Agents_ designed for platforms _without native HEP support._
<!-- tabs:start -->

## ** HEPlify **
<a id=heplify></a>

* [HEPlify](https://github.com/sipcapture/heplify): CA developed in go, portable, near zero configuration

### Installation

##### Linux
Download the [latest static build](https://github.com/sipcapture/heplify/releases) for any Linux system
```
curl -fsSL github.com/sipcapture/heplify/releases/latest/download/heplify -O
chmod +x fluxpipe-server
```

##### Windows
A windows build is available for [download](https://github.com/sipcapture/heplify/releases)

### Usage Examples

###### Capture SIP and RTCP packets on any interface and send them to 127.0.0.1:9060
```
./heplify
```
###### Capture SIP and RTCP packets on any interface and send them via TLS to 192.168.1.1:9060
```
./heplify -hs 192.168.1.1:9060 -nt tls
```
###### Capture SIP and RTCP packets on any interface and send them to 192.168.1.1:9060. Use a someNodeName
```
./heplify -hs 192.168.1.1:9060 -hn someNodeName
```
###### Capture SIP and RTCP packets on any interface and send them to 192.168.1.1:9060. Print info to stdout
```
./heplify -hs 192.168.1.1:9060 -e
```
###### Capture SIP and RTCP packets on any interface and send them to 192.168.1.1:9060 and 192.168.2.2:9060
```
./heplify -hs "192.168.1.1:9060,192.168.2.2:9060"
```
###### Capture SIP and RTCP packets on any interface and send them to 192.168.1.1:9060. Print debug selectors
```
./heplify -hs 192.168.1.1:9060 -e -d fragment,payload,rtcp
```
###### Capture SIP and RTCP packets with custom SIP port range on eth2 and send them to 192.168.1.1:9060
```
./heplify -i eth2 -pr 6000-6010 -hs 192.168.1.1:9060
```
###### Capture SIP and RTCP packets on eth2, send them to homer and compressed to /srv/pcapdumps/
```
./heplify -i eth2 -hs 192.168.1.1:9060 -wf /srv/pcapdumps/ -zf
```
###### Read example/rtp_rtcp_sip.pcap and send SIP and correlated RTCP packets to 192.168.1.1:9060
```
./heplify -rf example/rtp_rtcp_sip.pcap -hs 192.168.1.1:9060
```
###### Capture and send packets except SIP OPTIONS and NOTIFY to 192.168.1.1:9060.
```
./heplify -hs 192.168.1.1:9060 -dim OPTIONS,NOTIFY
```
###### Run heplify in "HEP Collector" mode in order to receive HEP input via TCP on port 9060 and fork (output) to two HEP servers listening on port 9063
```
./heplify -e -hs HEPServer1:9063,HEPserver2:9063 -hin tcp:1.2.3.4:9060
```

##### Parameters
The following parameters are available to configure your agent
```
 -assembly_debug_log
    	If true, the github.com/google/gopacket/tcpassembly library will log verbose debugging information (at least one line per packet)
  -assembly_memuse_log
    	If true, the github.com/google/gopacket/tcpassembly library will log information regarding its memory use every once in a while.
  -b int
    	Interface buffersize (MB) (default 32)
  -d string
    	Enable certain debug selectors [defrag,layer,payload,rtp,rtcp,sdp]
  -dd
    	Deduplicate packets
  -di string
    	Discard uninteresting packets by any string
  -dim string
    	Discard uninteresting SIP packets by CSeq [OPTIONS,NOTIFY]
  -disip string
    	Discard uninteresting SIP packets by Source IP(s)
  -e	Log to stderr and disable syslog/file output
  -erspan
    	erspan
  -fg uint
    	Fanout group ID for af_packet
  -fi string
    	Filter interesting packets by any string
  -fw int
    	Fanout worker count for af_packet (default 4)
  -hi uint
    	HEP node ID (default 2002)
  -hin
     HEP collector listening protocol, address and port (example: "tcp:10.10.99.10:9060")
  -hn string
    	HEP node Name
  -hp string
    	HEP node PW
  -hs string
    	HEP server destination address and port (default "127.0.0.1:9060")
  -i string
    	Listen on interface (default "any")
  -l string
    	Log level [debug, info, warning, error] (default "info")
  -lp int
    	Loop count over ReadFile. Use 0 to loop forever (default 1)
  -m string
    	Capture modes [SIP, SIPDNS, SIPLOG, SIPRTCP] (default "SIPRTCP")
  -n string
    	Log filename (default "heplify.log")
  -nt string
    	Network types are [udp, tcp, tls] (default "udp")
  -o	Read packet for packet
  -p string
    	Log filepath (default "./")
  -pr string
    	Portrange to capture SIP (default "5060-5090")
  -protobuf
    	Use Protobuf on wire
  -rf string
    	Read pcap file
  -rs
    	Use packet timestamps with maximum pcap read speed
  -rt int
    	Pcap rotation time in minutes (default 60)
  -s int
    	Snaplength (default 8192)
  -sl
    	Log to syslog
  -t string
    	Capture types are [pcap, af_packet] (default "pcap")
  -tcpassembly
    	If true, tcpassembly will be enabled
  -tcpsendretries uint
    	Number of retries for sending before giving up and reconnecting (default 64)
  -version
    	Show heplify version
  -vlan
    	vlan
  -wf string
    	Path to write pcap file
  -zf
    	Enable pcap compression
```      



## ** CaptAgent **
<a id=captagent></a>

* [CaptAgent](https://github.com/sipcapture/captagent): CA developed in C, ideal for complex configurations
  
### Installation

This section provides guidance to download the latest code from our repository and compile it on your system.


#### Requirements

* Captagent requires ```libuv``` 

If your system does not provide ```libuv``` and ```libuv-dev```, please install from our 
[repository](https://github.com/sipcapture/captagent/tree/master/dependency) or compile it from [source]( https://github.com/libuv/libuv/releases)

##### Operating Systems
###### Debian 10/11
```
apt-get install libexpat-dev libpcap-dev libjson-c-dev libtool automake flex bison libgcrypt-dev libuv1-dev libpcre3-dev libfl-dev
```
###### Debian 9 (stretch):
```
apt-get install libexpat1-dev libpcap-dev libjson-c-dev libtool automake flex bison libgcrypt11-dev libuv1-dev libpcre3-dev libfl-dev
```

###### CentOS 7/8:
```
yum -y install json-c-devel expat-devel libpcap-devel flex-devel automake libtool bison libuv-devel flex
```


#### Clone & Compile
```
  cd /usr/src
  git clone https://github.com/sipcapture/captagent.git captagent
  cd captagent
  ./build.sh
  ./configure
  make && make install
```

##### Build Options

| Name        | Configure Flag       | Libraries           |
|---          |---                   |---                  |
| HEP Compression | --enable-compression |                     |
| IPv6 Support  | --enable-ipv6        |                     |
| PCRE Support  | --enable-pcre        | libpcre             |
| SSL Support   | --enable-ssl         | openssl             |
| TLS Support   | --enable-tls         | libgcrypt20 openssl |
| MySQL Support | --enable-mysql       | libmysqlclient      |
| Redis Support | --enable-redis       | libhiredis          |

--------------


Congratulations! You just installed your first instance of CaptAgent!
    
<!-- tabs:end -->

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>
