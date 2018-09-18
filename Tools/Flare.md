Flare
========
Abstract
---------
Flare is a python script designed to identify command and control beaconing. It does this by analyzing flow data stored in Elasticsearch. It should support flow data from various data sources but currently has only been tested with Suricata flow logs.

**Background:** adversaries like to maintain access to compromised networks and systems. To do this, they often establish a command and control network. Infected systems periodically check in with command and control servers (also known as bot herders)

**Problem:** detecting connections to command and control servers is extremely hard. Connections may occur only once a day, every X seconds, or on a seemingly random connection pattern

**Solution:** reach out to Austin Taylor and Flare is born

Flare helps us solve the problem by applying automated analysis of flow data.


Where to Acquire
---------
Flare can be downloaded from https://github.com/austin-taylor/flare. 

Examples/Use Case
---------
This is an example configuration to run flare against an Elasticsearch index called lab5.1-complete-suricata:
```bash
[beacon]
es_host=localhost
es_index=lab5.1-complete-suricata
es_port=9200
es_timeout=480
min_occur=10
min_percent=50
window=2
threads=8
period=26280
kibana_version=4
verbose=True

#Elasticsearch fields for beaconing
field_source_ip=source_ip
field_destination_ip=destination_ip
field_destination_port=destination_port
field_timestamp=@timestamp
field_flow_bytes_toserver=bytes_to_server
field_flow_id=flow_id
```

This is the command to run Flare:
```bash
flare_beacon -c /labs/lab5.3/files/lab5.3.ini  --focus_outbound --whois --group --html=/labs/lab5.3/student/beacons.html
```