Kibana
========
Abstract
---------
Kibana is a report engine designed for Elasticsearch. It is a major open source component of the Elastic Stack. It is used to search data and visualize your logs through charts and tables as well as dashboards.

Where to Acquire
---------
Kibana can be downloaded from https://www.elastic.co/products/kibana. It is open source but also has a commercial support offering.

Search Filters
---------
Below are some of the common search filters used with Kibana.

This is an example of looking for an logs that contain the string "password":
```bash
password
```

This is an example of looking for logs that contain the name jhenderson stored in a field called user:
```bash
user:jhenderson
```

Note: Sometimes a string needs to be surrounded with double quotes.

Example:
```bash
"sec555.com"
```

This is an example of looking for logs that contain a source port greater than 40000:
```bash
source_port:>40000
```

This is an example of looking for logs that contain a destination IP between 10.0.0.0 and 10.255.255.255:
```bash
destination_ip:[10.0.0.0 TO 10.255.255.255]
```

This is an example of looking for logs that have a field named tls:
```bash
_exists_:tls
```

This is an example of looking for logs that do not have a field named tls:
```bash
_missing_:tls
```

This is an example of looking for logs that do not have a tag of pci:
```bash
-tags:pci
```

This is an example of looking for logs that are between a specific date:
```bash
@timestamp:[2017-05-01 TO 2017-05-28]
```

#### Combining search filters

Search filters can be combined using (), AND, and OR

This is an example of looking for a network connection sourcing from 192.168.0.1 going to 8.8.8.8:
```bash
source_ip:192.168.0.1 AND destination_ip:8.8.8.8
```

This is an example of looking for a network connection coming from 192.168.0.1 or 192.168.0.2:
```bash
source_ip:192.168.0.1 OR source_ip:192.168.0.2
```

This is an example of looking for a network connection coming from 192.168.0.1 or 192.168.0.2 that is destined for 8.8.8.8:
```bash
(source_ip:192.168.0.1 OR source_ip:192.168.0.2) AND destination_ip:8.8.8.8
```

This is an example of looking for network connections coming from 192.168.0.1 that are not going to 8.8.8.8:
```bash
source_ip:192.168.0.1 AND -destination_ip:8.8.8.8
```

Note: Using AND is not required when using an exclusion filter

Here is the same example as above that still works:
```bash
source_ip:192.168.0.1 -destination_ip:8.8.8.8
```

This is an example of looking for network connections that are not going to a private IP address:
```bash
-destination_ip:[10.0.0.0 TO 10.255.255.255] -destination_ip:[192.168.0.0 TO 192.168.255.255] -destination_ip:[172.16.0.0 TO 172.16.31.255.255]
```