<img src="https://user-images.githubusercontent.com/1423657/55069501-8348c400-5084-11e9-9931-fefe0f9874a7.png" width=120/><br>

# __HOMER__ Setup
This document provides guidance, packages and details to get HOMER installed & running

### HOMER 10 _(beta)_
* 🐋 [Docker Install](https://github.com/sipcapture/homer-docker) _(easiest)_


### HOMER 7
* 🎁 [Manual Install](#-manual-install) _(easy)_
* 📑 [Quick Install](#-quick-install) _(easier)_
* 🐋 [Docker Install](#-docker-install) _(easiest)_


#### Clients & Integrations
* 📞 [HEP Capture Agents](#-capture-agents)
* 📈 [HEP Grafana Integration](#-grafana-integration)



_Found an problem with this guide? Please open an [issue](https://github.com/sipcapture/homer/issues) and help us improve!_

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>

## 🎁 Manual Install:
Homer and its components can easily be installed manually in just minutes.

### Debian
Install the `sipcapture `repository for Debian/Ubuntu:
```
curl -s https://packagecloud.io/install/repositories/qxip/sipcapture/script.deb.sh | sudo bash
```
Install the latest package
###### HOMER/HEP Agent
  * `apt install heplify`
###### HOMER/HEP Server
  * `apt install heplify-server`
  * `apt install homer-app`


### CentOS
Install the `sipcapture `repository for CentOS/EL 7:
```
curl -s https://packagecloud.io/install/repositories/qxip/sipcapture/script.rpm.sh | sudo bash
```

###### HOMER/HEP Agent
  * `yum install heplify`
###### HOMER/HEP Server
  * `yum install heplify-server`
  * `yum install homer-app`


#### 🔧 Configuration
  * Configure Homer Capture Server
    `/opt/heplify-server/heplify-server.toml`
    * [Example Configuration](https://github.com/sipcapture/heplify-server/blob/master/example/homer7_config/heplify-server.toml)
    * Configure DB connectors
      * Postgres
      * Loki _(optional)_
      * Prometheus Metrics Endpoint
  * Configure Homer Application 
    `/usr/local/homer/etc/webapp_config.json`
    * [Example Configuration](https://github.com/sipcapture/homer-app/blob/master/etc/webapp_config.json)
    * Configure DB connectors
      * Postgres
      * Prometheus API _(optional)_
      * InfluxDB API _(optional)_
      * Loki API _(optional)_

#### ⏳ Initialization
Adjust with your database credentials, and execute the following commands to setup the backend components:

##### Create Homer DBs
```
homer-app -create-config-db -database-root-user=postgres -database-host=localhost -database-root-password=postgres -database-homer-user=homer_user
homer-app -create-data-db -database-root-user=postgres -database-host=localhost -database-root-password=postgres -database-homer-user=homer_user
```

##### Create Tables, Populate defaults, Upgrade
```
homer-app -create-table-db-config 
homer-app -populate-table-db-config 
homer-app -upgrade-table-db-config 
```

#### 🔖 Start Services
```
systemctl start heplify-server
systemctl start homer-app
```

#### Done!
You should now be able to access your HOMER instance via HTTP on port `9080` and send HEPv3 traffic to port `9060/UDP` or `9061/TCP` _(note ports can be modified by the configuration)_

![image](https://user-images.githubusercontent.com/1423657/73536888-5513dd80-4427-11ea-82aa-b2ce53192a63.png)

<br>
<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>

##  📑 Quick Install:
A script is provided to deploy the baseline elements with minimal interaction on a vanilla server:
[https://github.com/sipcapture/homer-installer](https://github.com/sipcapture/homer-installer)

Quick install currently supports modern `debian` and `centos` based operating systems.

### Instructions
Make sure the script is executed as `root` on a `netinstall` vanilla server:

for Debian
```
apt-get install libluajit-5.1-common libluajit-5.1-dev lsb-release wget curl git
```
For CentOS
```
yum install redhat-lsb-core wget curl git
```
then Download Homer Installer script
```
wget https://github.com/sipcapture/homer-installer/raw/master/homer_installer.sh
chmod +x homer_installer.sh
./homer_installer.sh
```


<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>

## 🐋 Docker Install:
A set of `docker-compose` bundles is provided to bootstrap a full Homer 7.7 deployment including optional elements:
[https://github.com/sipcapture/homer7-docker](https://github.com/sipcapture/homer7-docker/tree/7.7/heplify-server)

### Instructions
This procedure requires `docker` and `docker-compose` installed on the target system.

#### Deployment
```
git clone https://github.com/sipcapture/homer7-docker
cd homer7-docker/heplify-server/hom7-prom-all
docker-compose up -d
```
#### Access & Usage
  * Homer:9080 _(admin/sipcapture)_
  * Grafana:9030 _(admin/admin)_
  * Prometheus:9090 _(admin/admin)_
  * Loki:3100 _(admin/admin)_
  * Alertmanager:9093 _(admin/admin)_

#### Done!
You should now be able to access your HOMER instance via HTTP on port `9080` and send HEPv3 traffic to port `9060/UDP` or `9061/TCP` _(note ports can be modified by the docker-compose configuration)_

![image](https://user-images.githubusercontent.com/1423657/73536888-5513dd80-4427-11ea-82aa-b2ce53192a63.png)

<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>

## 📞 Capture Agents
Ready to ship data to Homer? Install a HEP Capture Agent based on your needs and preferences:
* [HEPlify](https://github.com/sipcapture/heplify):
  CA developed in go, portable, near zero configuration
* [CaptAgent](https://github.com/sipcapture/captagent): 
  CA developed in C, ideal for complex configurations
  
* **Native**:
  [Native HEP Agents](https://github.com/sipcapture/homer/wiki) are available in [OpenSIPS](https://github.com/sipcapture/homer/wiki/Examples%3A-OpenSIPS), [Kamailio](https://github.com/sipcapture/homer/wiki/Examples%3A-Kamailio), [Asterisk](https://github.com/sipcapture/homer/wiki/Examples%3A-Asterisk), [Freeswitch](https://github.com/sipcapture/homer/wiki/Examples%3A-FreeSwitch), [RTP:Engine](https://github.com/sipcapture/homer/wiki/Examples%3A-RTPEngine) and many more. Consult the Wiki to get specific examples for your platform.

* **Event Agents**:
HOMER can collect, index and correlate non-packet events such as Logs, RTC stats, CDRs, and more using [HEP](https://github.com/sipcapture/hep) supported by a variety of tools such as [paStash](https://github.com/sipcapture/pastash/wiki) and [Telegraf](https://github.com/influxdata/telegraf/pull/6167)


<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>

## 📈 Grafana Integration
Deploying HOMER alongside Grafana? 

Import the [HOMER Preset Dashboards & Widgets](https://grafana.com/grafana/dashboards?direction=asc&orderBy=name&search=homer) to get started with standard KPIs in seconds.

<img src="https://user-images.githubusercontent.com/20154956/39880524-57838c04-547e-11e8-8dec-262184192742.png">


<img src="https://camo.githubusercontent.com/0c29c4b70ff4b2958555ae30d3885eb4c34e5878/687474703a2f2f692e696d6775722e636f6d2f39414e303861752e676966"/>


Made with ❤️ by Humans

*(C) 2008-2021 [QXIP BV](http://qxip.net)*

----------

##### If you use HOMER in production, please consider supporting the project with a [Donation](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=donation%40sipcapture%2eorg&lc=US&item_name=SIPCAPTURE&no_note=0&currency_code=EUR&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHostedGuest)

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=donation%40sipcapture%2eorg&lc=US&item_name=SIPCAPTURE&no_note=0&currency_code=EUR&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHostedGuest)
