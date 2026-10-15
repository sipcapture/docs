# SONUS SBC

The Sonus SBC does not (yet) provide dynamic means for always-on packet monitoring except switch-based port mirroring/spanning. The only available resource to fetch information from is ```weblog.log``` which can be to some extend parsed and recycled using [paStash](https://github.com/sipcapture/pastash)

#### WARNING! This example setup is a workaround and might be unreliable!
## Requirements

* [HOMER](http://sipcapture.org) or [HEPIC](http://hepic.tel)
* SONUS SBC w/ weblog enabled
* HOST w/ NodeJS, npm

## Installation
The first step is to install paStash on our middleware HOST
```
# git clone https://github.com/sipcapture.pastash
# cd pastash
# sudo npm install
```

## Configuration
Configuration involves three stages: ```input```, ```filter```, ```output```

Adjust the following parameters as needed:
* input > file > path
* output > hep > host, port

Save the config file locally, ie: ```/opt/pastash/sonus.json```

```
input {
  file {
    path => "/var/log/sonus/webui.log"
# When testing with a static file:
#    start_index => 0
  }
}

filter {
  multiline {
    start_line_regex => /^\[\d{4}-\d{2}-\d{2}/
  }
  grok {
    match => '(?m)\[%{TIMESTAMP_ISO8601:timestamp}\] %{WORD:pid} %{WORD:seq} \n%{GREEDYDATA:payload}'
  }
  regex {
    regex => /From:.*\@(.*)\:(\d+)/
    fields => [srcIp,srcPort]
  }
  regex {
    regex => /To:.*\@(.*)\:(\d+)/
    fields => [dstIp,dstPort]
  }
  regex {
    regex => /Call-I.*: (.*)/
    fields => [correlation_id]
  }
}

output {
  if [tags] != "_grokparsefailure" {
        hep {
          host => '127.0.0.1'
          port => 9063
          hep_id => 2222
          hep_type => 1
        }
  }
}
```

## HEP Time
It's time to run our recipe and check for results:
```
./bin/pastash --config_file=/opt/pastash/sonus.json
```

If everything is configured correctly and wind is on our side, paStash should start convertnig relevant Sonus logs to HEP-SIP packets over UDP. NGREP to confirm this is true before searching for received data.

--------

### Known Issues
* Fragmented messages are not yet handled
* IP/PORT extracted from signaling, unreliable source
* no caching from loglines preceding the SIP methods
* only tested by a few users, needs feedback & tuning
