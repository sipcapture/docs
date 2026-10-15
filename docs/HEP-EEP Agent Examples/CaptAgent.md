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
  
#### Next: [Configure CaptAgent 6](https://github.com/sipcapture/captagent/wiki/Configuration)
