## Abstract
CaptAgent has been around for a while and went through several redesign phases.

The current version (6) has been completely redesigned from the ground up and ships with new internal architecture expanding its performance range and core capabilities to handle more network protocols, the ability to relay and aggregate traffic for remote agents, parse and handle RTCP-XR and RTP statistics, RPC centralized control and much more in a modular architecture.

## Internal Architecture
##### Sockets, Pipes and Plans

Captagent 6 features a fully modular design enabling users to design and program their packet capture and processing logic, leveraging specialized functionality provided on-demand via loadable dynamic modules.

Core module types include:

| type  | description  |
|:--|:--|
| socket  | responsible for capturing ingress packets according to settings _(ie: PCAP, RAW)_ |
| protocol  | responsible for processing/dissecting/parsing protocol data _(ie: SIP, RTCP)_ |
| transport  | responsible for providing egress transport for generated data _(ie: HEP, JSON)_|
| function  | responsible for providing additional functionality _(ie: database, etc)_ |

Core modules are loaded via the main ```captagent.xml``` configuration file and can be easily concatenated to create multiple, independent capture chains:

<img src="http://i.imgur.com/GalHujZ.png">

In the above example:

```SOCKET``` -> ```PROFILE``` -> ```CAPTURE PLAN``` <--> ```MODULES (functions)```

-------------


###### CAPTURE CHAINS
For each chain, the logic and functionality is managed using a "capture-plan" which defines the behavior of the packet processing pipe. Capture plans are defined within the socket configuration alongside the general capture settings. An example for PCAP socket follows:
```
            <settings>
		<param name="dev" value="any"/>
		<param name="promisc" value="true"/>
		<param name="reasm" value="false"/>
		<param name="capture-plan" value="sip_capture_plan.cfg"/>
		<param name="filter">
		    <value>portrange 5060-5091</value>
		</param>
	    </settings>
```

In the above example, packets captured by the socket would be processed by *capture-plan* in  ```sip_capture_plan.cfg```:

```
# PCAP socket module
capture[pcap] {
        # PROTO SIP module
	# Ie: check source/destination IP/port, message size, etc.
	if(msg_check("size", "100")) {
	    # Parse SIP Protocol
	    if(parse_sip()) {
		# use HEP TRANSPORT module (transport_hep.xml)	
		if(!send_hep("hepsocket")) {
		    clog("ERROR", "Error sending HEP!");
		}
            }
       }
}
```

The capture-plan can access all functions provided by the loaded modules globally.


### Main Features
* Multiple incoming sockets (PCAP, RAW, PF_RING, RX-RING, FILE)
* Multiple outgoing types (HEP, JSON, CSV)
* HTTP JSON API for statistics, config changes etc
* RTCP-XR collector module
* RTCP output module (output in raw or json format)
* Capture scenario configuration (pseudo scripting via flex, bison)
* Call transaction tracking
* TCP/UDP reassembling and defragmentation.
* applying and change capture filter on demand
* LUA scripting (JITLua) (experimental)
* V7 Javascripting sandbox (experimental)
* SIPFIX Support (experimental)
* Websocket encapsulation support

## CaptAgent: Installation

This section provides guidance to download the latest code from our repository and compile it on your system.


### Requirements

* Captagent 6.2+ requires ```libuv``` 

    If your system does not provide ```libuv``` and ```libuv-dev```, please install from our 
[repository](https://github.com/sipcapture/captagent/tree/master/dependency) or compile it from [source]( https://github.com/libuv/libuv/releases)

##### Operating Systems
###### Debian 10 (buster):
```
apt-get install libexpat-dev libpcap-dev libjson-c-dev libtool automake flex bison libgcrypt-dev libuv1-dev libpcre3-dev libfl-dev

```
###### Debian 9 (stretch):
```
apt-get install libexpat1-dev libpcap-dev libjson-c-dev libtool automake flex bison libgcrypt11-dev libuv1-dev libpcre3-dev libfl-dev

```
###### Debian 8 (jessie):
```
apt-get install libexpat-dev libpcap-dev libjson0-dev libtool automake flex bison libuv-dev libgcrypt11-dev libfl-dev
```
###### Debian 7 (wheezy):
```
wget https://github.com/sipcapture/captagent/raw/master/dependency/debian/wheezy/libuv_1.8.0-2_amd64.deb
dpkg -i libuv_1.8.0-2_amd64.deb

apt-get install libexpat-dev libpcap-dev libjson0-dev libtool automake flex bison 

```

###### CentOS 7:
```
yum -y install json-c-devel expat-devel libpcap-devel flex-devel automake libtool bison libuv-devel flex
```

###### CentOS 6:
```
rpm -i https://github.com/sipcapture/captagent/raw/master/dependency/centos/6/libuv-1.8.0-1.el6.x86_64.rpm
rpm -i https://github.com/sipcapture/captagent/raw/master/dependency/centos/6/libuv-devel-1.8.0-1.el6.x86_64.rpm
yum -y install json-c-devel expat-devel libpcap-devel pcre-devel flex-devel automake libtool bison flex
```



### Clone & Compile
```
  cd /usr/src
  git clone https://github.com/sipcapture/captagent.git captagent
  cd captagent
  ./build.sh
  ./configure
  make && make install
```

#### Build Options
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

#### TLS Support (experimental)

To compile and enable TLS decryption features, please check the dedicated Wiki page.

--------------


Congratulations! You just installed your first basic instance of CaptAgent 6!
  
#### Next: [Configure CaptAgent 6](#captagent-configuration)


# Captagent Init Scripts

The init.d script can be used to start/stop captagent in a nicer way. 

### Installation & Configuration

A sample of init.d script for captagent is provided at:
```
/usr/src/captagent/init/deb/debian/captagent.init
```

Just copy the init file to ```/etc/init.d/captagent``` and change the permisions:

```
  cp /usr/src/captagent/init/deb/debian/captagent.init /etc/init.d/captagent
  chmod 755 /etc/init.d/captagent 
```

then edit the file updating the $DAEMON and $CFGFILE values:

```
  DAEMON=/usr/local/captagent/bin/captagent
  CFGFILE=/usr/local/captagent/etc/captagent/captagent.xml
```

You need also setup a configuration file in the /etc/default/ directory:
```
  cp /usr/src/captagent/init/deb/debian/captagent.default /etc/default/captagent
```

When using systemd _(we all wish we were not)_ the service file might be required:
```
  cp /usr/src/captagent/init/deb/debian/captagent.service /etc/systemd/system/captagent.service
```

Once installed and before using, edit the default file to reflect your captagent path:
```
   RUN_CAPTAGENT=yes
   CFGFILE=/usr/local/captagent/etc/captagent/captagent.xml
```


### Automatic Startup
To execute automatically at startup:
```
   update-rc.d captagent defaults
```

To exclude the script from startup:
```
   update-rc.d -f  captagent remove
```


## CaptAgent: Configuration

This section provides guidance to configure Captagent and its core modules on your system.

#### Configuration Logic

Understanding of CaptAgent configuration structure is key - read [this](#internal-architecture) section carefully!


-------------

#### Basic Configuration

The following are the default file locations _(unless otherwise specified during configuration)_:

* Configuration: ```/usr/local/captagent/etc```
* Capture Plans: ```/usr/local/captagent/etc/captureplans```
* Modules: ```/usr/local/captagent/lib/modules```


##### Configuration tree
The default directory should contains the following using default profiles and plans:
```
captagent.xml
captureplans/
    sip_capture_plan.cfg
    rtcp_capture_plan.cfg
    rtcpxr_capture_plan.cfg
    tzsp_capture_plan.cfg
protocol_rtcp.xml
protocol_sip.xml
protocol_rtcpxr.xml
protocol_diameter.xml
socket_pcap.xml
socket_tzsp.xml
socket_collector.xml
transport_hep.xml
output_json.xml

```

##### Main Configuration
To begin, edit and validate the configuration and the module paths in ```/usr/local/etc/captagent/captagent.xml``` to match your actual captagent config/lib path:

```
<configuration name="core.conf" description="CORE Settings" serial="2014024212">
            <settings>
                <param name="debug" value="3"/>
                <param name="version" value="2"/>
                <param name="serial" value="2014056501"/>
                <param name="uuid" value="00781a4a-5b69-11e4-9522-bb79a8fcf0f3"/>
                <param name="daemon" value="false"/>
                <param name="syslog" value="false"/>
                <param name="pid_file" value="/var/run/captagent.pid"/>
                <param name="module_path" value="/usr/local/lib/captagent/modules"/>
                <param name="config_path" value="/usr/local/etc/captagent"/>
                <param name="capture_plans_path" value="/usr/local/etc/captagent/captureplans"/>
                <param name="backup" value="/usr/local/etc/captagent/backup"/>
                <param name="chroot" value="/var/lib/captagent"/>
            </settings>
        </configuration>
```

### Next:

* Configure [Socket Modules](#socket-modules)
* Configure [Transport Modules](#transport-modules)
* Configure [Capture Plans](#capture-plans)

### Socket Modules
Socket modules are used by captagent to capture packets from the system available interfaces. By default, the ```pcap``` socket is enabled.

#### PCAP Socket
Within each socket, multiple profiles can be defined and configured as pipelines to process captured packets. By default the ```socketspcap_sip``` pipeline is enabled defining the interface and ports utilized for capturing and processing ```SIP``` packets via the dedicated [Capture Plan](#capture-plans)

```
      <profile name="socketspcap_sip" description="HEP Socket" enable="true" serial="2014010402">
	    <settings>
		<param name="dev" value="eth0"/>
		<param name="promisc" value="true"/>
		<param name="reasm" value="false"/>
                <param name="websocket-detection" value="false"/>
		<param name="tcpdefrag" value="false"/>
                <!-- <param name="capture-filter" value="ip_to_ip"/> -->
		<param name="capture-plan" value="sip_capture_plan.cfg"/>
		<param name="filter">
		    <value>portrange 5060-5091</value>
		</param>
	    </settings>
	</profile>
```
### GRE-ERSPAN Example
When capturing GRE-ERSPAN Encapsulated traffic this needs to be setup
```
    <param name="erspan" value="true"/>
    <param name="filter">
       <value>proto GRE and len > 50</value>
    </param>
```

### Websocket encapsulation layer
Sometimes WebSocket subprotocol it is used as a reliable transport mechanism between Session Initiation Protocol (SIP) entities to enable use of SIP in web-oriented deployments. 

Captagent provide additional parsing for Websocket layer on TCP by enable the `websocket-detection` param (default is false)
```
   <param name="websocket-detection" value="true"/>
```

### IP-to-IP encapsulation layer
`IP-to-IP` is an IP tunneling protocol that encapsulates one IP packet in another IP packet. To encapsulate an IP packet in another IP packet, an outer header is added with source IP, the entry point of the tunnel, and the destination IP, the exit point of the tunnel.

Captagent now has the possibility to check and correctly parse this tunnel by adding a new `capture-filter` on portrange `5060-590`:
```
   <param name="capture-filter" value="ip_to_ip"/>
```
This filter perform also the internal check on portrange 5060-5090, so in the BPF filter you can simply add this
```
   <param name="filter">
       <value>ip</value>
   </param>
```

### Transport Modules
Transport modules are used by captagent to send packets and reports to collectors using different methods and protocols. By default, the *HEP* method is activated.

#### HEP
The *HEP* module is used to define a HEP collector for captured packets, such as [HOMER](http://github.com/sipcapture/homer/wiki).

The critical parameters are:

* capture-host: defines the IP/hostname of the collector
* capture-port: defines the PORT to deliver HEP packets at the collector
* capture-proto: defines the transport protocol for HEP packets [udp/tcp]
* capture-id: defines a unique delivery HEP-ID to be used for filtering

NOTE: Parameters such as ```capt-password``` and ```payload-compression``` are currently only used in advanced deployments and can be ignored for standard setups.

##### Example HEP Configuration:
```
<?xml version="1.0"?>
<document type="captagent_module/xml">
    <module name="transport_hep" description="HEP Protocol" serial="2014010402">
        <profile name="hepsocket" description="Transport HEP" enable="true" serial="2014010402">
            <settings>
                <param name="version" value="3"/>
                <param name="capture-host" value="your.homer.ip"/>
                <param name="capture-port" value="9060"/>
                <param name="capture-proto" value="udp"/>
                <param name="capture-id" value="2016"/>
                <param name="capture-password" value="myHep"/>
                <param name="payload-compression" value="false"/>
            </settings>
        </profile>
    </module>
</document>
```



#### JSON
The *JSON* module is used to define a collector used by captagent to deliver JSON messages and statistics, such as [HOMER](http://github.com/sipcapture/homer/wiki).

The critical parameters are:

* capture-host: defines the IP/hostname of the JSON API
* capture-port: defines the PORT for the JSON API
* capture-proto: defines the transport protocol for HEP packets [udp|tcp]
* capture-id: defines a unique delivery HEP-ID to be used for filtering
* payload-send: defines if full packet payload should be included in API POST [true|false]

##### Example JSON Configuration:

```
<?xml version="1.0"?>
<document type="captagent_module/xml">
    <module name="output_json" description="JSON Protocol" serial="2014010402">
	<profile name="jsonsocket" description="Output JSON" enable="true" serial="2014010402">
	    <settings>
		<param name="capture-version" value="1"/>
		<param name="capture-host" value="127.0.0.1"/>
		<param name="capture-port" value="9061"/>
		<param name="capture-proto" value="udp"/>
		<param name="capture-id" value="2001"/>
		<param name="payload-send" value="true"/>
	    </settings>
	</profile>
    </module>
</document>
```


### Protocol Modules
Protocol modules are used by captagent to process specific protocol pipelines. 
Protocol modules are loaded at startup by the ```captagent.xml``` general configuration.


```
<configuration name="modules.conf" description="Modules">
	    <modules>
	
		...
		<load module="protocol_sip" register="local"/>
		<load module="protocol_rtcp" register="local"/>		                                  
		...
		
	    </modules>
</configuration>
```

### Functions to use in modules
#### protocol_sip

**Note:** This list is probably incomplete by the time you read it. You can get all functions available here: [protocol_sip.c](https://github.com/sipcapture/captagent/blob/master/src/modules/protocol/sip/protocol_sip.c)

| Function | Description |
| ---| --- |
| `sip_check("method", "INVITE")` | only INVITE requests |
| `sip_check("rmethod", "REGISTER")` | REGISTER requests and answers (CSeq) |
| `sip_check("response", "200")` | only 200 responses                |
| `sip_check("response_gt", "400")` | only responses >= 400          |
| `sip_check("response_lt", "199")` | only responses <= 199          |
| `sip_check("from_user_suffix", "tail")` |  check if from_user has "tail" as suffix   |
| `sip_check("to_user_suffix", "tail")` |  check if to_user has "tail" as suffix   |
| `sip_check("from_user_prefix", "head")` |  check if from_user has "head" as prefix   |
| `sip_check("to_user_prefix", "head")` |  check if to_user has "head" as prefix   |
| `msg_check("size", "100")` | only packets > 100 bytes              |
| `msg_check("src_ip", "10.0.0.1")` | only packets from 10.0.0.1     |
| `msg_check("destination_ip", "10.0.0.1")` | only packets to 10.0.0.1 |
| `msg_check("src_port", "10000")` | only packets from port 10000    |
| `msg_check("src_port_gt", "10000")` | only packets from ports >= 10000 |
| `msg_check("src_port_lt", "10000")` | only packets from ports <= 10000 |
| `msg_check("dst_port", "10000")` | only packets to port 10000      |
| `msg_check("dst_port_gt", "10000")` | only packets to ports >= 10000 |
| `msg_check("dst_port_lt", "10000")` | only packets to ports <= 10000 |
| `sip_is_method()` | check whether this is a request                |
| `parse_sip()` | check for valid SIP message                        |
| `light_parse_sip()` | check whether Call-ID Header is valid        |
| `parse_full_sip()` | check for valid SIP and parse Contact and Via header |
| `sip_has_sdp()` | check whether sip packet has sdp                 |
| `is_flag_set("1", "0")` | check whether flag named "1" has value "0" |
| `send_reply("200", "OK")` | generate a reply from Captagent        |
| `clog("ERR", "foo")` | generate error log line                     |
| `clog("NOTICE", "foo")` | generate notice log line                 |
| `clog("foo", "bar")` | generate debug log line                     |


## WARNING!
*This is a work in Progress! Issues must be raised w/ full details + PCAP to reproduce.*

## Captagent TLS
When provided with the appropriate keying material, the TCP protocol module can attempt decryption TLS  connections and display the application data traffic in real-time. 

##### How decryption work 
Internally, the work is divided to resolve the _asymmetric_ and _symmetric_ encryption.
1. The _encrypted pre-master secret_ is captured during the key exchange between Client and Server, and decrypted to obtain a **Pre-Master Secret (PMS)**.
2. The **Master Secret (MS)** is recreated thanks to _PMS_, and used for the symmetric encryption of the keys. The _MS_ is necessary to build the _keys block_, that is dissected and leveraged to regenerate the required decryption keys used in the asymmetric part of the encryption. 
3. These keys (**MACs** or **IVs**, and **Write** keys) perform the final decryption of the _real_ data.

Decryption can only be attempted for scenarios including the full client-server handshake.

##### SUPPORTED:
* TLS_RSA_WITH_AES_256_GCM_SHA384
* TLS_RSA_WITH_AES_128_GCM_SHA256

  * RSA_PKCS1_PADDING

##### UNSUPPORTABLE:
* TLS_DH* (Diffie-Hellman)

-------------

### Requirements

* RSA PRIVATE KEY (max size _2048_ bit)
* libssl-dev
* libgcrypt 1.8 or higher

#### Debian
```
apt-get install -y libgcrypt20 libgcrypt20-dev libssl-dev
```

### Compile & Install
```
./build.sh
./configure --enable-tls --enable-ssl
make && sudo make install
```

### Configure

#### captagent.xml
Enable loading for ```protocol_tls``` under `/usr/local/captagent/etc/captagent/captagent.xml`
```
           [...]
            <load module="protocol_tls" register="local"/>    <---- our module
            <load module="socket_pcap" register="local"/>
           [...]
```
Remember to put `<load module="protocol_tls" register="local"/>` **before** `<load module="socket_pcap" register="local"/>` in order to activate the protocol functions in the correct way.

#### socket_pcap
Configure ```socket_pcap``` and enable the TLS profile block using the proper port or portrange:
```
<profile name="socketspcap_tls" description="TLS Socket" enable="true" serial="2014010402">
	    <settings>
		<param name="dev" value="any"/>
		<param name="promisc" value="true"/>
		<param name="reasm" value="false"/>
		<param name="tcpdefrag" value="true"/>
		<param name="capture-plan" value="tcp_capture_plan.cfg"/>
		<param name="filter">
		    <value>tcp port 5061</value>
		</param>
	    </settings>
	</profile>
```

#### protocol_tls
Configure ```protocol_tls``` with the full path to the required ```private-key``` to decrypt RSA/TLS traffic:
```
<?xml version="1.0"?>
<document type="captagent_module/xml">
  <module name="protocol_tls" description="TLS Protocol" serial="2014010402">
    <profile name="proto_tls" description="TLS PROTO" enable="true" serial="2014010402">
      <settings>
	<param name="flow-timeout" value="180"/>
	<!-- the value of private key refers to the absolute path of the private key (used for decryption) -->
	<param name="private-key-path" value="/path/to/cakey.pem"/>
      </settings>
    </profile>
  </module>
</document>
```

_That's all!_

### Dev Demo
Development [demo setup](https://github.com/lmangani/docker-opensips-hepclient/tree/tls) leverages the default [OpenSIPS rootCA](https://github.com/OpenSIPS/opensips/tree/master/etc/tls/rootCA).

##### TODO
For a complete scenarios it will be desirable to have other supported cipher suites for the RSA family

### Troubleshooting
The code is relatively fresh and might be impacted by TCP reassembly and many other challenging scenarios requiring fine-tuning. If your traffic is not decoded or you receive TLS parsing errors for traffic you can decode with Wireshark, please open and issue w/ full details + PCAP to reproduce.


### Capture Plans
Capture Plans are pipelines attached to capture sockets and utilized to define processing logic, 
using global functions and methods exported by the loaded modules as defined in ```captagent.xml```.

The socket is defined in each capture plan, supporting the following `capture[...]` types:

* pcap
* tzsp
* collector

##### Example Configuration Chain
```socket_pcap``` -> ```{profile}``` -> ```capture_plan```

##### Example Pointer
/usr/local/etc/captagent/socket_pcap.xml
```
        <profile name="socketspcap_sip" description="HEP Socket" enable="true" serial="2014010402">
	    <settings>
		<param name="dev" value="eth0"/>
		<param name="promisc" value="true"/>
		<param name="reasm" value="false"/>
		<param name="tcpdefrag" value="false"/>
		<param name="capture-plan" value="sip_capture_plan.cfg"/>
		<param name="filter">
		    <value>portrange 5060-5091</value>
		</param>
	    </settings>
	</profile>
```

##### Example Capture Plan
/usr/local/etc/captagent/captureplans/sip_capture_plan.cfg
```
capture[pcap] {
	# here we can check source/destination IP/port, message size
	if(msg_check("size", "100")) {
	    #Do parsing
	    if(parse_sip()) {
                # Drop unwanted methods
                if(sip_check("rmethod","OPTIONS") || sip_check("rmethod","NOTIFY")) {
                  drop;
                } 

		#Multiple profiles can be defined in transport_hep.xml	
		if(!send_hep("hepsocket")) {
		    clog("ERROR", "Error sending HEP!!!!");
		}
	    }
	}
	drop;
}
```


In order for captagent to correlate and pair SIP SDP Sessions to RTCP packets, the following configuration block should be uncommented in the ```sip_capture_plan.cfg```:
```
if(sip_has_sdp())
	{
		#Activate it for RTCP checks
		if(!check_rtcp_ipport())
		{
			clog("ERROR", "SDP SESSION ALREADY EXISTS!");
		}
	}
```

This will produce a match for the ```is_rtcp_exist()``` function in the ```rtcp_capture_plan.cfg``` scope to inject a correlation ID and return results in HOMER.

## NAT
When the SDP details are reporting masqueraded or private IP ranges, the ```nat-mode``` switch can be enabled to force Captagent to use the received address for matching media sessions. Settings enabled by module ```database_hash``` should be applied in file ```database_hash.xml``` :

```
<?xml version="1.0"?>
<document type="captagent_module/xml">
    <module name="database_hash" description="HASH Database" serial="2014010402">
	<profile name="database_hash" description="HASH RTCP" enable="true" serial="2014010402">
	    <settings>
		<param name="timer-expire" value="80"/>
		 <!-- enable if you have nat devices and masqaraded ip -->
		 <param name="nat-mode" value="true"/>		    
	    </settings>
	</profile>
    </module>
</document>
```


