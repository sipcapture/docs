Guys, please if you have some issues, provide a little bit more information. We couldn't guess or read your mind remotely. 

## Latest versions:
First of all, please be sure that you have the latest version of homer-app / homer-ui / heplify-server:

[Homer-APP](https://github.com/sipcapture/homer-app/releases/latest)

[Homer-UI](https://github.com/sipcapture/homer-ui/releases/latest)

[Heplify-Server](https://github.com/sipcapture/heplify-server/releases/latest)

## version desync : homer-app vs homer-ui
The homer-app is an API server (backend). The homer-ui is an UI frontend (JS/HTML). The frontend communicates with the homer-API by using JSON/Rest protocol. The packages (rpm/deb) of homer-app contain the actual version of UI (minimized and gziped). If you have installed the homer-app (binary) manualy (download from a repo or compiled from the sources), this means you have to upgrade/install the homer-ui also manualy (check how it to do using sources, go to https://github.com/sipcapture/homer-ui). Once you have compiled it, copy the dist files to the dist directory of homer-app, usualy it's /usr/local/homer/dist, (to be sure, check your webapp_conf.json: https://github.com/sipcapture/homer-app/blob/master/etc/webapp_config.json#L48). If you are lazy or don't know how to use npm to compile javascript, you can download the latest compiled version of the frontend:  https://github.com/sipcapture/homer-app/releases/latest .


1.1.32 - an example here. Please use the latest version!
```
wget https://github.com/sipcapture/homer-app/releases/download/1.1.32/homer-ui-7.7.028.tgz
tar xzf homer-ui-7.7.028.tgz
cp -Rp dist/* /usr/local/homer/dist/

```



or use our packagecloud.io repository (https://github.com/sipcapture/homer/wiki/Quick-Install#-manual-install)

So. You run the latest versions and you still have issues, than you go to next steps:

**If you are upgrading Homer 7.0 to Homer 7.7. Please reinit your configuration database:**

`./homer-app -populate-table-db-config -force-populate`

or if you have some users and don't loose them, upgrade only these tables:

`./homer-app -populate-table-db-config -force-populate -populate-table=mapping_schema -populate-table=global_settings -populate-table=user_settings -populate-table=hepsub_mapping_schema`

***
## Database inspection:

**Be sure you have some SIP (or RTCP, LOG) data in the tables of your DB (homer_data)** 

Execute in the shell (terminal/ssh) of the DB host:

```
# su - postgres
$ psql homer_data
homer_data=# \dt
```

it will show you if you have any tables in your database:

**I don't see any tables, the DB is empty or psql tells you:  "psql: FATAL:  database "homer_data" does not exist"**

- you didn't install postgress or it's not up.
- you didn't create database (see the https://github.com/sipcapture/homer/wiki/Quick-Install#-initialization)
- you didn't run heplify-server or there is any issues with connectivity (wrong credentials, a wrong path to the config of heplify-server ( heplify-server.toml ) or any firewall/iptable, because your heplify-server will create the tables on first start and will do the partition rotation job.

**Oh, I have tables:**

Great, now:

**Check if you have any data inside:**

```
select count(*) from hep_proto_1_call;
  count  
---------
 3993340
(1 row)

```
***
## HEP inspection:

if you see 0 rows = you didn't send any HEP data or again wrong configuration (heplify-server, captagent or firewall) Be sure that you send 
- **SIP**
-  encapsulation protocol **HEPv3** , not HEPv2

Again **non RTCP non LOG, you send SIP** 

(if you are not sure - capture packets with wireshark/ngrep/tcpdump (ports: 9060 or 9063) and do analyze with https://github.com/sipcapture/hep-wireshark . In worse case, check if you have HEPv3 (ASCII) at the begin of packet's payload)

***
## Debug in web console
**if you have records, but you don't see anything in your UI, but you can login and create widgets, you see configurations etc...**

in the browser, press **F12** (Web console):

and do a search again. Please be sure that you have selected the correct time range.
If you don't see any requests in the Network tab:

**in Homer - UI do:**

- Logout 
- Login 
- and press Search again

and NOW:

**Check the /api/v3/search/call/data 
Don't execute this query, go to the Network tab in your Web console F12 (see the screenshoots)
and check the reply of this api request and analyze the data. Check if you have some records inside** 

*The screenshoots:**

![Screenshot from 2020-02-13 09-06-48](https://user-images.githubusercontent.com/4513061/74415509-cbf7a000-4e43-11ea-9a45-18b0c8f6977e.png)

![Screenshot from 2020-02-13 09-07-40](https://user-images.githubusercontent.com/4513061/74415533-d7e36200-4e43-11ea-8cea-8d2421c6e191.png)

***
## UI/APP misconfiguration

**if you don't see any data in the result grid (table) of UI, but you have data in hep_proto_1_call (DB:  homer_data, Table: hep_proto_1_call):** :

- timerange is wrong - you don't have any data for this time range. Be sure you don't have any search parameters and make the range bigger.
- the  database connection to homer_data in your webapp_conf.json is wrong or firewall issue.__

## Submit a bug report:

**If you see a response with DATA inside, but nothing has been displayed:**

Congratulation, probably you found a bug, and you can submit an issue
- Submit versions of your homer-app/UI/heplify/heplify-server  (the version of UI you can find in the title.)
- How did you install the applications (package / installer script / self compiling)
- How do you send data to Homer (SIP/ RTCP/ RTP / LOG)
- Small network topology description
- any additonal information that can help us understand and solve your issue


I hope this small instruction helps you to localize your problem and avoid waste your and our time for a ping-pong communication.

