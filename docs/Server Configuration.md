##### HEP Socket for UDP Traffic
The address for the HEP server to listen on for HEP traffic.
```
HEPAddr               = "0.0.0.0:9060"
```

##### HEP Socket for TCP Traffic
The TCP address for the HEP server. (For now its empty, indicating it is not being used)
```
HEPTCPAddr            = "0.0.0.0:9062"
```

##### HEP Socket for TLS Traffic
The TLS address for the HEP server to listen on for secure HEP traffic.
```
HEPTLSAddr            = "0.0.0.0:9063"
```

##### Elasticsearch Endpoint Address
The address for Elasticsearch.
```
ESAddr                = "http://127.0.0.1:9200"
```
##### Elasticsearch Endpoint discovery
A boolean value indicating whether Elasticsearch discovery is enabled or not.
```
ESDiscovery           = true
```

##### Loki Endpoint Address
The URL for Loki, a log aggregation system.
```
LokiURL               = "http://localhost:3100/api/prom/push"
```
##### Loki Bulk Max Size
The number of log entries to send to Loki in bulk.
```
LokiBulk              = 200
```
##### Loki Bulk Timeout
The interval (in seconds) at which logs are sent to Loki.
```
LokiTimer             = 4
```
##### Loki Buffer size
The maximum number of log entries that can be buffered before sending to Loki.
```
LokiBuffer            = 100000
```
##### Loki HEP Filter
A list of HEP filter values to specify which types of HEP messages should be sent to Loki.
```
LokiHEPFilter         = [1,5,100]
```
##### Loki HEP Payload
A list of HEP payload types to force their inclusion in the HEP messages.
```
ForceHEPPayload         = []
```
##### Prometheus Endpoint Address
The address for Prometheus.
```
PromAddr              = "0.0.0.0:8899"
```
##### Prometheus Target IP
The IP address for the Prometheus target.
```
PromTargetIP          = "10.1.2.111,10.1.2.4,10.1.2.5,10.1.2.6,10.12.44.222"
```
##### Prometheus Target Name
The name of the Prometheus target.
```
PromTargetName        = "sbc_access,sbc_core,kamailio,asterisk,pstn_gateway"
```
##### Database Schema (homer7, homer5)
The database schema to use.
```
DBShema               = "homer7"
```
##### Database Driver (postgres)
The database driver to use (in this case, "postgres").
```
DBDriver              = "postgres"
```
##### Database Address
The address of the database server. In this case, it is set to "localhost:5432", indicating that the database is located on the local machine and listening on port 5432.
```
DBAddr                = "localhost:5432"
```
##### Database Auth User
The username used to authenticate with the database server. In this case, it is set to "postgres".
```
DBUser                = "postgres"
```
##### Database Auth Password
The password used to authenticate with the database server. Currently empty, indicating that no password is provided.
```
DBPass                = ""
```
##### Database Data Table name
The table name in the database where HEP data will be stored. In this case, it is set to "homer_data".
```
DBDataTable           = "homer_data"
```
##### Database Config Table name
The table name in the database where HEP configuration will be stored. In this case, it is set to "homer_config".
```
DBConfTable           = "homer_config"
```
##### Database Bulk max size
The number of database entries to insert in bulk. It is set to 200, meaning that database entries will be inserted in groups of 200.
```
DBBulk                = 200
```
##### Database Bulk Timeout
The interval (in seconds) at which database entries are inserted. 
```
DBTimer               = 4
```
##### Database Buffer Size
The maximum number of database entries that can be buffered before inserting. 
```
DBBuffer              = 400000
```
##### Database Worker Instances
The number of worker threads for handling database operations.
```
DBWorker              = 8
```
##### Rotation Enable
A boolean value indicating whether database table rotation is enabled or not.
```
DBRotate              = true
```
##### Database Rotation Settings
```
DBPartLog             = "2h"
This param specifies the duration for which the log data should be stored in the database. 

DBPartSip             = "1h" 
This param determines the duration for which SIP data should be stored in the database.

DBPartQos             = "6h"
This param specifies the duration for which data related to  Quality of Service (QoS) should be stored in the database.
```
##### Database Drop Settings
```
DBDropDays            = 14
This param specifies the number of days after which all data in the database (except logs) will be deleted automatically. 

DBDropDaysCall        = 0
This param sets the number of days after which call-related data will be deleted from the database.

DBDropDaysRegister    = 0
This param indicate the number of days after which registration-related data will be dropped from the database.

DBDropDaysDefault     = 0
This param specifies the number of days after which any other data that is not related to the above categories will be dropped from the database.

DBDropOnStart         = false
This param specifies the number of days after which all data in the database (except logs) will be deleted automatically. Users must use this action carefully as it is irreversible. 
```
##### Enable Packet De-Duplication
This param enables or disables deduplication of packets in the database.
```
Dedup                 = false
```
##### Discard Methods (array)
This param specifies the methods for discarding data that is not captured by the server. 
```
DiscardMethod         = ["OPTIONS","NOTIFY"]
```
##### Custom Headers
```
AlegIDs               = ["X-CID","P-Charging-Vector,icid-value=\"?(.*?)(?:\"|;|$)","X-BroadWorks-Correlation-Info"]
This param allows you to specify A-leg IDs to filter the captured data. Only data related to the specified A-leg IDs will be stored in the database.

CustomHeader          = ["X-CustomerIP","X-Billing"]
 This param is used to define custom headers.
```
##### Logger Settings
```
LogDbg                = "hep,sql,loki"
This param controls the debug log configuration.

LogLvl                = "info"
This param determines the log level for the server. 

LogStd                = false
A param whether logs should be output to the standard output.

LogSys                = false
A param indicates whether logs should be sent to the system logger.
```
##### Configuration Location
This param specifies the path to the configuration file for the server.
```
Config                = "./heplify-server.toml"
```
##### Configuration Web Editor URL
This param determines the HTTP address for the configuration server. 
```
ConfigHTTPAddr        = "0.0.0.0:9876"
```
