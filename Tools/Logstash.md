Logstash
========
Abstract
---------
Logstash is a log aggregator designed to collect, parse, and enrich logs. It is a major open source component of the Elastic Stack. It can be used in conjunction with other commerial SIEM solutions.

Where to Acquire
---------
Logstash can be downloaded from https://www.elastic.co/products/logstash. It is open source but also has a commercial support offering.

Examples/Use Case
---------
Below are some of the common configurations used with Logstash.

General Knowledge
---------
- [type](#type)
- [tags](#tags)
- [conditional logic](#logic) - If statements
<!-- - [architecture](#architecture) -->

---------
### type
The **type** field is a special field often used to control how logs are handled. It can be used to control how a log is parsed and ultimately stored.

Type is most commonly set within an input plugin. For example, this configuration would set the type to windows anytime logs come in over TCP port 6052:
```bash
input {
    tcp {
        port => 6052
        type => "windows"
    }
}
```
This type could then be used to conditionally interact with logs with a type of "windows":

```bash
filter {
  if [type] == "windows" {
      do something...
  }
}
```

It can also be used to control where the logs ultimately are saved such as an Elasticsearch index of logstash-windows-2017-06-10:
```bash
output {
    if [type] == "windows" {
        elasticsearch {
            index => "logstash-windows-%{+YYYY.MM.dd}"
        }
    }
}
```

---------
### tags
Tags are attributes used to apply conditional filtering or to ease searching. It also can be used to route logs to their final destination. A log can have an unlimited amount of tags.

Tags can be set at input but can also be applied within the filter section. Below is an example of setting a tag of "windows" for any logs coming in over port 6052:
```bash
input {
    tcp {
        port => 6052
        tags => "windows"
    }
}
```
Below is an example of adding a tag within the filter portion of Logstash:

```bash
filter {
  mutate {
      add_tag => "windows"
  }
}
```

Below is an example of adding multiple tags within the filter portion of Logstash:

```bash
filter {
  mutate {
      add_tag => [ "windows", "pci", "critical_asset" ]
  }
}
```

Tags can be used to control where the logs ultimately are saved such as an Elasticsearch index of logstash-windows-2017-06-10:
```bash
output {
    if "windows" in [tags] {
        elasticsearch {
            index => "logstash-windows-%{+YYYY.MM.dd}"
        }
    }
}
```

Tags can greatly aid in searching and detection techniques. This can be used so that an analyst can quickly search for whether a connection is outbound to the internet or to an internal system. It also can be used to apply log enrichment to select logs. 

For example, below is an example of tagging IP addresses. 
```bash
filter {
  if [destination_ip] =~ "2(?:2[4-9]|3\d)(?:\.(?:25[0-5]|2[0-4]\d|1\d\d|[1-9]\d?|0)){3}" {
    mutate {
      add_tag => [ "multicast" ]
    }
  }
  if [destination_ip] == "255.255.255.255" {
    mutate {
      add_tag => [ "broadcast" ]
    }
  }
  if [destination_ip] and "multicast" not in [tags] and "broadcast" not in [tags] {
    if [destination_ip] =~ "10\." or [destination_ip] =~ "192\.168\." or [destination_ip] =~ "172\.(1[6-9]|2[0-9]|3[0-1])\." {
      mutate {
        add_tag => [ "internal_destination" ]
      }
    } else {
      mutate {
        add_tag => [ "external_destination" ]
      }
    }
  }
}
```

---------
### logic

Conditional logic is required to properly scale and use Logstash in an enterprise environment. It allows you to accept, parse, or enrich logs based on specific conditions.

A simple condition would be to do something **IF** a value equals something specific. Example:
```javascript
filter {
    if [field] == "7" {
        do_something...
    }
}
```

#### Conditions dealing with Numbers

**Note:** That "7" is not the same as 7. **"7"** means the string of 7 and **7** means an integer of 7. If the field contained the integer 7 you would need to do this:
```javascript
filter {
    if [field] == 7 {
        do_something...
    }
}
```

If a field contains a number value than you can use greater than or less than. Examples:
```javascript
filter {
    if [field] > 7 {
        do_something...
    }
}
```

```javascript
filter {
    if [field] < 7 {
        do_something...
    }
}
```

You can also do less than or equal to and greater than or equal to. Example:
```javascript
filter {
    if [field] >= 7 {
        do_something...
    }
}
```

#### Conditions dealing with Strings

When checking against a string you can do an exact match such as follows:
```javascript
filter {
    if [field] == "string" {
        do_something...
    }
}
```

Or you can do a regex match such as below. The **=~** specifies a regex match. The example below looks for the string "string" followed by any characters.
```javascript
filter {
    if [field] =~ "string*" {
        do_something...
    }
}
```

#### Condition checks against a field's existance
If you want to apply a configuration but only if a field exists use this:
```javascript
filter {
    if [field] {
        do_something...
    }
}
```

The syntax **if [field] {** means only run if the field **field** exists.

#### Handling multiple conditions

Sometimes you need to apply multiple logical conditions. To do this use **and** and **or**. For example, the below configuration requires a specific string for **field1** and an integer over 5 for **field2**.
```javascript
filter {
    if [field1] == "string" and [field2] > 5 {
        do_something...
    }
}
```

This is a similar example but where **field1** needs to be set to string **or** **field2** needs to be over 5.

```javascript
filter {
    if [field1] == "string" or [field2] > 5 {
        do_something...
    }
}
```

Input Plugins
---------
- [beats](#beats)
- [elasticsearch](#elasticsearch)
- [file](#file)
- [jdbc](#jdbc)
- [tcp](#tcp)
- [udp](#udp)
- [rabbitmq](#rabbitmq)
- [kafka](#kafka)

Special considerations (not actual plugins)

- [codecs](#codecs)

---------
### beats
The **beats** plugin is an input plugin used to accept logs from beats agents such as winlogbeat and filebeat.

This example shows how to listen for logs from beats on port 5044. You cannot change the type when using beats.
```javascript
input {
    beats {
        port => 5044
    }
}
```

---------
### elasticsearch
The **elasticsearch** input plugin is an input plugin used to accept logs from an elasticsearch index. This can be used to re-import logs from an existing index. It requires a query to be specified.

```javascript
input {
    elasticsearch {
        hosts => "localhost"
        query => '{ "query": { "match": { "statuscode": 200 } }, "sort": [ "_doc" ] }'
    }
}
```

---------
### jdbc
The **jdbc** plugin is an input plugin used to retrieve logs from a database such as Microsoft SQL or MySQL. It is often used to retrieve logs from third party systems that store logs in a database such as McAfee ePO (endpoint protection suite).

```javascript
input {
    jdbc {
        jdbc_driver_library => "/etc/logstash/drivers/sqljdbc42.jar"
        jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
        jdbc_connection_string => "jdbc:sqlserver://sqlserver_name_goes_here:1433;databasename=database_name_goes_here"
        jdbc_fetch_size => "10000"
        jdbc_user => "sql_user_goes_here"
        jdbc_password => "sql_password_goes_here"
        schedule => "* * * * *"
        statement => "SELECT id,ipv4,user,mesesage FROM logs"
        tracking_column => id
        type => "mssql"
    }
}
```
---------
### file
The **file** plugin is an input plugin used to monitor files or folders.

Input configuration for monitoring a file:
```javascript
input {
    file {
        path => "/var/log/syslog"
    }
}
```
Input configuration for monitoring a folder:
```javascript
input {
    file {
        path => "/var/log/"
    }
}
```
Input configuration for monitoring CSV files within a folder:
```javascript
input {
    file {
        path => "/path/to/some/folder/*.csv"
    }
}
```
---------
### tcp
The **tcp** input plugin is an input plugin that listens on a TCP port for logs.

Input configuration for accepting logs on TCP port 1025:
```javascript
input {
    tcp {
        port => 1025
    }
}
```
It is recommended to add either a tag or type or both to all inputs. These should be used to specify the expected logs and/or information about these logs such as method of collection. This example shows both:
```javascript
input {
    tcp {
        port => 1025
        type => "name_goes_here"
        tags => "name_goes_here"
    }
}
```
On Linux operating systems you must have be root or have administrative privileges to listen on ports 1024 and below. However, running Logstash as root is a bad idea. One way around this is to use iptables. The iptables command below maps port 514 (syslog) to port 1514.
```bash
iptables -t nat -A PREROUTING -p tcp --dport 514 -j REDIRECT --to-port 1514
```
The equivalent Logstash configuration file that accepts these logs is below:
```javascript
input {
    tcp {
        port => 1514
        type => "syslog"
    }
}
```
To make iptables persist across reboots you can use iptables-save and iptables-restore. For more information on this see this link:
https://help.ubuntu.com/community/IptablesHowTo#Using_iptables-save.2Frestore_to_test_rules

### udp
The **udp** input plugin is an input plugin that listens on a UDP port for logs.

Input configuration for accepting logs on UDP port 1025:
```javascript
input {
    udp {
        port => 1025
    }
}
```
It is recommended to add either a tag or type or both to all inputs. These should be used to specify the expected logs and/or information about these logs such as method of collection. This example shows both:
```javascript
input {
    udp {
        port => 1025
        type => "name_goes_here"
        tags => "name_goes_here"
    }
}
```
On Linux operating systems you must have be root or have administrative privileges to listen on ports 1024 and below. However, running Logstash as root is a bad idea. One way around this is to use iptables. The iptables command below maps port 514 (syslog) to port 1514.
```bash
iptables -t nat -A PREROUTING -p udp --dport 514 -j REDIRECT --to-port 1514
```
The equivalent Logstash configuration file that accepts these logs is below:
```javascript
input {
    udp {
        port => 1514
        type => "syslog"
    }
}
```
To make iptables persist across reboots you can use iptables-save and iptables-restore. For more information on this see this link:
https://help.ubuntu.com/community/IptablesHowTo#Using_iptables-save.2Frestore_to_test_rules

### rabbitmq
The **rabbitmq** input plugin is an input plugin that retrieves logs stored in RabbitMQ, which is a common third party message broker/log buffer.

This example shows the basic rabbitmq settings needed by Logstash:
```javascript
input {
  rabbitmq {
    key => "logstashkey"
    queue => "logstashqueue"
    durable => true
    exchange => "logstashexchange"
    user => "logstash"
    password => "password_goes_here"
    host => "rabbitmq_server_goes_here"
    port => 5672
  }
}
```

This example below shows the basic RabbitMQ settings needed by Logstash but also includes some tags for troubleshooting. It assumes that it is pulling Windows logs out of a queue for Windows. The tags help troubleshoot issues related to a specific queue.
```javascript
input {
  rabbitmq {
    key => "logstashkey"
    queue => "windows"
    durable => true
    exchange => "logstashexchange"
    user => "logstash"
    password => "password_goes_here"
    host => "rabbitmq_server_goes_here"
    port => 5672
    tags => [ "queue_windows", "rabbitmq" ]
  }
}
```

### kafka
The **kafka** input plugin is an input plugin that retrieves logs stored in Kafka, which is a common third party message broker/log buffer.

This example shows the basic kafka settings needed by Logstash:
```javascript
input {
  kafka {
    zk_connect => "kafka_server_goes_here:2181"
    topic_id => [ "logstash" ]
  }
}
```

This example below shows the basic Kakfa settings needed by Logstash but also includes some tags for troubleshooting. It assumes that it is pulling Windows logs out of a queue for Windows. The tags help troubleshoot issues related to a specific queue.
```javascript
input {
  kafka {
    zk_connect => "kafka_server_goes_here:2181"
    topic_id => [ "windows" ]
    tags => [ "queue_windows", "kafka" ]
  }
}
```

---------
### codecs

Codecs can be used in input plugins to tell Logstash what data representation to expect in incoming logs. For example, if you know that logs coming in to the **tcp** plugin are going to be json you could use this:

```javascript
input {
    tcp {
        port => "6000"
        codec => "json"
    }
}
```

The above configuration would automatically extract json field data similar to the below configuration. The difference is the below configuration must first shove the log into a field called **message** and then extract the json information from it.

```javascript
input {
    tcp {
        port => "6000"
    }
}
filter {
    json {
        source => "message"
    }
}
```

Filter Parsing Plugins
---------
- [csv](#csv)
- [date](#date)
- [grok](#grok)
- [json](#json)
- [kv](#kv)
<!-- - [xml](#xml) -->

---------
### csv
The **csv** plugin is an filter plugin used to parse out fields that are seperated by a specific character.

The below example configuration retrieves specific columns that are tab delimited from a Bro DHCP log
```javascript
filter {
    csv {
      columns => ["timestamp","uid","source_ip","source_port","destination_ip","destination_port","protocol","transaction_id","rtt","query","query_class","query_class_name","query_type","query_type_name","rcode","rcode_name","aa","tc","rd","ra","z","answers","ttls","rejected"]
      separator => "	"
    }
}
```

---------
### date
The **date** plugin is an filter plugin used to parse and normalize the date from a given field.

Date uses the match parameter to set patterns to parse out timestamps. The match parameter takes an array of one or more patterns. For more information see: https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-match

The below example configuration attempts to match a syslog time format against a field called syslog_timestamp.
```javascript
filter {
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
}
```

The below example configuration attempts to match a syslog time format against a field called syslog_timestamp and includes a timezone.
```javascript
filter {
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      timezone => "America/Chicago"
    }
}
```

The below example configuration attempts to match a timestamp based on the traditional year-month-day hour:minute:second format such as 2017-07-06 04:30:00.
```javascript
filter {
    date {
      match => ["EventTime", "YYYY-MM-dd HH:mm:ss"]
    }
}
```

The below example configuration attempts to match a timestamp based on the traditional UNIX timestamp format format such as 1496031788.649121 (taken from Bro log).
```javascript
filter {
    date {
      match => ["timestamp", "UNIX"]
    }
}
```

The below example configuration attempts to match a timestamp based ISO8601 format which is 2017-07-06T04:30:00.100Z.
```javascript
filter {
    date {
      match => ["EventTime", "ISO8601"]
    }
}
```

When using the date plugin, it may make sense to remove the original date field after it has been converted successfully. This can be done by adding remove_field such as below.
```javascript
filter {
    date {
      match => ["EventTime", "ISO8601"]
      remove_field => [ "EventTime"]
    }
}
```

---------
### grok
The **grok** plugin is an filter plugin used to parse fields using patterns and regex.

Grok allows both raw regex and grok patterns (which are really regex anyway) to be used in any combination. It is an extremely powerful tool.

When building out grok parsers it may make sense to do so using the website Grok Debugger:

https://grokdebug.herokuapp.com/

Patterns are simple to apply.

Take this sentence:
**The dog is brown and 4 years old.**

This would be the way to parse out the type of animal, color, and age using patterns:
```javascript
filter {
    grok {
        match => { "message" => "The %{WORD:animal} is %{WORD:color} and %{INT:age} years old." }
    }
}
```

This would be the way to parse out the type of animal, color, and age using regex:
```javascript
filter {
    grok {
        match => { "message" => "The (?<animal>[a-zA-Z]+) is (?<color>[a-zA-Z]+) and (?<age>[0-9]+) years old." }
    }
}
```

Please keep in mind that sometimes special characters will need escaped with "\". For example, consider this message:

There are [10] items in storage.

This grok configuration would fail:
```javascript
filter {
    grok {
        match => { "message" => "There are [%{INT:number}] items in storage." }
    }
}
```

This would be the correct configuration that escapes the [ and ] characters:
```javascript
filter {
    grok {
        match => { "message" => "There are \[%{INT:number}\] items in storage." }
    }
}
```

Now, what if a log sometimes has extra fields and other times does not? This would cause the pattern/regex match to fail. The way around this is to either perform multiple grok matches or handle optional fields.

Consider you need grok to be able to parse both of these sentences:

**The dog is brown and 4 years old.**
**The dog is brown and 4 years old and is owned by George.**

####**Multiple grok match**

Multiple grok matches simply need multiple match statements. The order of operations matters. The first match stops grok unless the parameter break_on_match is set to FALSE.

This is an example of a grok configuration that can parse both the sentences above using multiple grok match statements:
```javascript
filter {
    grok {
        match => { "message" => "The %{WORD:animal} is %{WORD:color} and %{INT:age} years old and is owned by %{WORD:owner}." }
        match => { "message" => "The %{WORD:animal} is %{WORD:color} and %{INT:age} years old." }
    }
}
```
####**Handling optional fields**

It is not uncommon for logs to have optional fields that sometimes exist and other times do not. To handle this with grok you can simply specific something as optional by surrounding it in ()?.

This is an example of a grok configuration that can parse both sentences above by adding an optional field reference using ()? in the configuration:
```javascript
filter {
    grok {
        match => { "message" => "The (?<animal>[a-zA-Z]+) is (?<color>[a-zA-Z]+) and (?<age>[0-9]+) years old( and is owned by %{WORD:owner})?." }
    }
}
```

Here is an example of grok being used against a Linux auth.log event:

```bash
<11>Jun 10 22:45:01 sec-555-linux CRON[2385]: pam_unix(cron:session): session closed for user root
```

The below example configuration attempts to parse the log above with grok patterns.
```javascript
filter {
    grok {
        match => { "message" => "<%{INT:syslog_pri}>%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    }
}
```

This is an example of looking for base64 text in a Windows PowerShell event ID 4104:
```javascript
filter {
    if [event_id] == 4104 and [ScriptBlockText] and [source_name] == "Microsoft-Windows-PowerShell" {
        grok {
            match => { "ScriptBlockText" => "(?<possible_base64_code>[A-Za-z0-9+/]{50,}[=]{0,2})" }
            tag_on_failure => []
        }
    }
}
```

Setting tag_on_failure to [] tells grok to not add a tag of _grokparsefailure if it fails to find a match.

Common Grok Patterns are below:

```bash
USERNAME [a-zA-Z0-9._-]+
USER %{USERNAME}
INT (?:[+-]?(?:[0-9]+))
BASE10NUM (?<![0-9.+-])(?>[+-]?(?:(?:[0-9]+(?:\.[0-9]+)?)|(?:\.[0-9]+)))
NUMBER (?:%{BASE10NUM})
BASE16NUM (?<![0-9A-Fa-f])(?:[+-]?(?:0x)?(?:[0-9A-Fa-f]+))
BASE16FLOAT \b(?<![0-9A-Fa-f.])(?:[+-]?(?:0x)?(?:(?:[0-9A-Fa-f]+(?:\.[0-9A-Fa-f]*)?)|(?:\.[0-9A-Fa-f]+)))\b

POSINT \b(?:[1-9][0-9]*)\b
NONNEGINT \b(?:[0-9]+)\b
WORD \b\w+\b
NOTSPACE \S+
SPACE \s*
DATA .*?
GREEDYDATA .*
QUOTEDSTRING (?>(?<!\\)(?>"(?>\\.|[^\\"]+)+"|""|(?>'(?>\\.|[^\\']+)+')|''|(?>`(?>\\.|[^\\`]+)+`)|``))
UUID [A-Fa-f0-9]{8}-(?:[A-Fa-f0-9]{4}-){3}[A-Fa-f0-9]{12}

# Networking
MAC (?:%{CISCOMAC}|%{WINDOWSMAC}|%{COMMONMAC})
CISCOMAC (?:(?:[A-Fa-f0-9]{4}\.){2}[A-Fa-f0-9]{4})
WINDOWSMAC (?:(?:[A-Fa-f0-9]{2}-){5}[A-Fa-f0-9]{2})
COMMONMAC (?:(?:[A-Fa-f0-9]{2}:){5}[A-Fa-f0-9]{2})
IPV6 ((([0-9A-Fa-f]{1,4}:){7}([0-9A-Fa-f]{1,4}|:))|(([0-9A-Fa-f]{1,4}:){6}(:[0-9A-Fa-f]{1,4}|((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){5}(((:[0-9A-Fa-f]{1,4}){1,2})|:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){4}(((:[0-9A-Fa-f]{1,4}){1,3})|((:[0-9A-Fa-f]{1,4})?:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){3}(((:[0-9A-Fa-f]{1,4}){1,4})|((:[0-9A-Fa-f]{1,4}){0,2}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){2}(((:[0-9A-Fa-f]{1,4}){1,5})|((:[0-9A-Fa-f]{1,4}){0,3}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){1}(((:[0-9A-Fa-f]{1,4}){1,6})|((:[0-9A-Fa-f]{1,4}){0,4}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(:(((:[0-9A-Fa-f]{1,4}){1,7})|((:[0-9A-Fa-f]{1,4}){0,5}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:)))(%.+)?
IPV4 (?<![0-9])(?:(?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2}))(?![0-9])
IP (?:%{IPV6}|%{IPV4})
HOSTNAME \b(?:[0-9A-Za-z][0-9A-Za-z-]{0,62})(?:\.(?:[0-9A-Za-z][0-9A-Za-z-]{0,62}))*(\.?|\b)
HOST %{HOSTNAME}
IPORHOST (?:%{HOSTNAME}|%{IP})
HOSTPORT (?:%{IPORHOST=~/\./}:%{POSINT})

# paths
PATH (?:%{UNIXPATH}|%{WINPATH})
UNIXPATH (?>/(?>[\w_%!$@:.,-]+|\\.)*)+
TTY (?:/dev/(pts|tty([pq])?)(\w+)?/?(?:[0-9]+))
WINPATH (?>[A-Za-z]+:|\\)(?:\\[^\\?*]*)+
URIPROTO [A-Za-z]+(\+[A-Za-z+]+)?
URIHOST %{IPORHOST}(?::%{POSINT:port})?
# uripath comes loosely from RFC1738, but mostly from what Firefox
# doesn't turn into %XX
URIPATH (?:/[A-Za-z0-9$.+!*'(){},~:;=@#%_\-]*)+
#URIPARAM \?(?:[A-Za-z0-9]+(?:=(?:[^&]*))?(?:&(?:[A-Za-z0-9]+(?:=(?:[^&]*))?)?)*)?
URIPARAM \?[A-Za-z0-9$.+!*'|(){},~@#%&/=:;_?\-\[\]]*
URIPATHPARAM %{URIPATH}(?:%{URIPARAM})?
URI %{URIPROTO}://(?:%{USER}(?::[^@]*)?@)?(?:%{URIHOST})?(?:%{URIPATHPARAM})?

# Months: January, Feb, 3, 03, 12, December
MONTH \b(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:tember)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?)\b
MONTHNUM (?:0?[1-9]|1[0-2])
MONTHDAY (?:(?:0[1-9])|(?:[12][0-9])|(?:3[01])|[1-9])

# Days: Monday, Tue, Thu, etc...
DAY (?:Mon(?:day)?|Tue(?:sday)?|Wed(?:nesday)?|Thu(?:rsday)?|Fri(?:day)?|Sat(?:urday)?|Sun(?:day)?)

# Years?
YEAR (?>\d\d){1,2}
HOUR (?:2[0123]|[01]?[0-9])
MINUTE (?:[0-5][0-9])
# '60' is a leap second in most time standards and thus is valid.
SECOND (?:(?:[0-5][0-9]|60)(?:[:.,][0-9]+)?)
TIME (?!<[0-9])%{HOUR}:%{MINUTE}(?::%{SECOND})(?![0-9])
# datestamp is YYYY/MM/DD-HH:MM:SS.UUUU (or something like it)
DATE_US %{MONTHNUM}[/-]%{MONTHDAY}[/-]%{YEAR}
DATE_EU %{MONTHDAY}[./-]%{MONTHNUM}[./-]%{YEAR}
ISO8601_TIMEZONE (?:Z|[+-]%{HOUR}(?::?%{MINUTE}))
ISO8601_SECOND (?:%{SECOND}|60)
TIMESTAMP_ISO8601 %{YEAR}-%{MONTHNUM}-%{MONTHDAY}[T ]%{HOUR}:?%{MINUTE}(?::?%{SECOND})?%{ISO8601_TIMEZONE}?
DATE %{DATE_US}|%{DATE_EU}
DATESTAMP %{DATE}[- ]%{TIME}
TZ (?:[PMCE][SD]T|UTC)
DATESTAMP_RFC822 %{DAY} %{MONTH} %{MONTHDAY} %{YEAR} %{TIME} %{TZ}
DATESTAMP_OTHER %{DAY} %{MONTH} %{MONTHDAY} %{TIME} %{TZ} %{YEAR}

# Syslog Dates: Month Day HH:MM:SS
SYSLOGTIMESTAMP %{MONTH} +%{MONTHDAY} %{TIME}
PROG (?:[\w._/%-]+)
SYSLOGPROG %{PROG:program}(?:\[%{POSINT:pid}\])?
SYSLOGHOST %{IPORHOST}
SYSLOGFACILITY <%{NONNEGINT:facility}.%{NONNEGINT:priority}>
HTTPDATE %{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} %{INT}

# Shortcuts
QS %{QUOTEDSTRING}

# Log formats
SYSLOGBASE %{SYSLOGTIMESTAMP:timestamp} (?:%{SYSLOGFACILITY} )?%{SYSLOGHOST:logsource} %{SYSLOGPROG}:
COMMONAPACHELOG %{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
COMBINEDAPACHELOG %{COMMONAPACHELOG} %{QS:referrer} %{QS:agent}

# Log Levels
LOGLEVEL ([A-a]lert|ALERT|[T|t]race|TRACE|[D|d]ebug|DEBUG|[N|n]otice|NOTICE|[I|i]nfo|INFO|[W|w]arn?(?:ing)?|WARN?(?:ING)?|[E|e]rr?(?:or)?|ERR?(?:OR)?|[C|c]rit?(?:ical)?|CRIT?(?:ICAL)?|[F|f]atal|FATAL|[S|s]evere|SEVERE|EMERG(?:ENCY)?|[Ee]merg(?:ency)?)
```

---------
### json
The **json** plugin is an filter plugin used to automatically extract fields from a json log.

This is an example of a JSON log:
```javascript
{"Hostname":"nessus01.sec555.com","Keywords":-9223372036854775808,"Severity":"INFO","ProviderGuid":"{C2E6D0D9-5DF8-4C77-A82B-C96C84579543}","Version":0,"Task":1,"Domain":"NT AUTHORITY","Message":"Unloading the management provider","Opcode":"Stop","EventData":"","@version":"1","@timestamp":"2017-05-27T02:27:51.000Z","host":"172.16.0.2","port":54451,"type":"windows","tags":[],"user":"Network Service","account_type":"Well Known Group","category":"Provider initialization","channel":"Microsoft-Windows-ServerManager-MgmtProvider/Operational","event_id":2,"event_received_time":1495852073,"event_type":"INFO","opcode_value":2,"process_id":2396,"record_number":2605,"severity_value":2,"source_module_name":"eventlog","source_module_type":"im_msvistalog","source_name":"Microsoft-Windows-ServerManager-ManagementProvider","thread_id":4468,"logstash_time":0.0}
```
Notice that it is surrounded by { } and contains "field":"value". JSON can also handle nested fields such as:

```javascript
{"Name":"Justin Henderson",
"Email":"justin@hasecuritysolutions.com",
"Attributes": {
    "EyeColor":"blue",
    "Height":"average"
}}
```
JSON is extremely easy to parse because you do not really parse it. You simply use the json plugin to automatically extract all the fields including nested ones.

The default source field is message. You only need to specific the source field if it is not message. Here is an example configuration that will automatic extract fields from the message field using json:
```javascript
filter {
    json { }
}
```

Here is an example configuration that will automatic extract fields from the event field using json:
```javascript
filter {
    json {
        source => "event"
    }
}
```

---------
### kv
The **kv** plugin is an filter plugin used to automatically extract fields from a key value based log.

Key value means data is stored using a field name, some kind of separator character, and the field value.

Example of key value data separated by = (default and most common)
```bash
source_ip=192.168.0.1 source_port=50000 destination_ip=8.8.8.8 destination_port=53
```

This is an example of a real kv log:
```bash
<189>date=2017-04-23 time=21:21:46 devname=FGT50E3U16006093 devid=FGT50E3U16006093 logid=0001000014 type=traffic subtype=local level=notice vd=root srcip=192.168.254.2 srcport=123 srcintf=root dstip=208.91.112.51 dstport=123 dstintf=wan1 sessionid=16237216 proto=17 action=accept policyid=0 dstcountry=Canada srccountry=Reserved trandisp=noop service=NTP app=NTP duration=181 sentbyte=76 rcvdbyte=76 sentpkt=1 rcvdpkt=1 appcat=unscannedroot
```

The configuration below uses kv to automatically extract fields and their corresponding values:

```javascript
filter {
    grok {
        match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    }
    kv {
        source => "syslog_message"
    }
}
```

The above configuration first uses grok to parse out the traditional syslog fields. It stores the syslog message into a field called syslog_message and then uses kv against it. This means kv was ran only against this:

```bash
devname=FGT50E3U16006093 devid=FGT50E3U16006093 logid=0001000014 type=traffic subtype=local level=notice vd=root srcip=192.168.254.2 srcport=123 srcintf=root dstip=208.91.112.51 dstport=123 dstintf=wan1 sessionid=16237216 proto=17 action=accept policyid=0 dstcountry=Canada srccountry=Reserved trandisp=noop service=NTP app=NTP duration=181 sentbyte=76 rcvdbyte=76 sentpkt=1 rcvdpkt=1 appcat=unscannedroot
```

Sometimes a log will come over with an empty value. In order for kv to work, the value portion cannot be empty.

For example, this is acceptable:
```bash
source_ip:192.168.0.1 vd=root destination_ip=8.8.8.8
```

This is not:
```bash
source_ip=192.168.0.1 vd= destination_ip=8.8.8.8
```

One way to handle this is to perform a string replacement of empty values and change the value from nothing to something like na (for not applicable). Here is an example on how to do this:
```javascript
filter {
    mutate {
        gsub => [ "syslog_message", "= ", "=na " ]
    }
    kv {
        source => "syslog_message"
    }
}
```

The above configuration will take this syslog_message:
```bash
source_ip=192.168.0.1 vd= destination_ip=8.8.8.8
```

Then it will convert it into this:
```bash
source_ip=192.168.0.1 vd=na destination_ip=8.8.8.8
```

Finally, it will use kv to automatically extract the following fields and values:

**source_ip**:192.168.0.1
**vd**:"na"
**destination_ip**:8.8.8.8


Filter Enrichment Plugins
---------
- [dns](#dns)
- [drop](#drop)
- [elasticsearch](#filter_elasticsearch)
- [memoize](#memoize)
- [geoip](#geoip)
<!-- - [jdbc_streaming](#jdbc_streaming) -->
- [mutate](#mutate)
- [rest](#rest)
- [ruby](#ruby)
- [syslog_pri](#syslog_pri)
- [tld](#tld)
- [translate](#translate)

---------
### dns
The **dns** plugin is a filter plugin used to either resolve a name to an IP address or an IP address to a name.

This example takes a domain and resolves it to an IP address:
```javascript
filter {
    dns {
        resolve => "hostname"
        action => "replace"
    }
}
```

This example takes an IP address and resolves it to a name:
```javascript
filter {
    dns {
        reverse => "source_ip"
        action => "replace"
    }
}
```

Both of the examples above would overwrite the fields specific (hostname and source_ip) with the value of the DNS answer. This is **not** the expected behavior. To get around this, add a field that clones the value of the original field and use the dns plugin against it.

Here is an example that first takes the value of the field hostname and puts it into a new field called hostname_resolved. Then it takes the hostname_resolved field and resolves it to an IP address:

```javascript
filter {
    mutate {
        add_field => { "hostname_resolved" => "%{hostname}" }
    }
    dns {
        resolve => "hostname_resolved"
        action => "replace"
    }
}
```

Here is an example that first takes the value of the field source_ip and puts it into a new field called source_ip_resolved. Then it takes the source_ip_resolved field and resolves it to a domain name:
```javascript
filter {
    mutate { add_field => { "source_ip_resolved" => "%{source_ip}" }
    }
    dns {
        reverse => "source_ip_resolved"
        action => "replace"
    }
}
```

---------
### drop
The **drop** plugin is a filter plugin used to remove an event. A dropped event is immediately removed from Logstash and not processed.

The **drop* plugin is extremely important. Use it to eliminate events that have little to no value.

This example drops an event if the field syslog_message has the phrase "unknown exception has occurred":
```javascript
filter {
    if [syslog_message] =~ "unknown exception has occurred" {
        drop { }
    }
}
```

This example drops an event if the EventID field is set to 555:
```javascript
filter {
    if [EventID] == 555 {
        drop { }
    }
}
```

---------
### filter_elasticsearch
The **elasticsearch** plugin is community filter plugin used to query Elasticsearch. If a match is found, whatever fields are specified will be appended to the existing log.

If lookups are likely to happen multiple times for the same piece of data consider using this plugin in conjunction to #memoize. It will enable caching of the results and allow Logstash to perform much faster lookups.

Consider this scenario, an IDS alert has been received but the alert only contains the source_ip and destination_ip. However, having the DNS name associated with these IP addresses could be valuable to an analyst. If DNS queries and answers are in Elasticsearch, they can be pulled into the IDS alert automatically.

Here is the configuration to do it that works with Logstash versions 5.x/6.x:
```javascript
filter {
    if [event_type] == "alert" {
        elasticsearch {
            query => "highest_registered_domain:%{highest_registered_domain}"
            index => "alexa-top1m"
            fields => {
                "rank" => "rank"
            }
        }
    }
}
```

Here is the configuration to do it that works with Logstash version 2.x:
```javascript
filter {
    if [event_type] == "alert" {
        elasticsearch {
            hosts => ["localhost"]
            index => "logstash-suricata-dns-*"
            query => "event_type:dns AND query_type:answer AND response_data:%{[destination_ip]}"
            fields => [["query_name","query"],["dns_id","dns_id"]]
        }
    }
}
```

This configuration would take the destination_ip and check to see if the Elasticsearch index logstash-suricata-dns-* had a previous answer that contained the destination_ip. If a match was found the elasticsearch filter plugin would pull back the query_name and dns_id fields. In this example, the query_name field would be stored into a field called query and the dns_id field would be stored into a field called dns_id.

---------
### memoize
The **logstash-filter-memoize** plugin is community filter plugin used to enable caching for other filter plugins. It is used to wrap itself around another filter plugin and cache the results based on a specific field. Subsequent calls to the same filter plugin with the same field value causes memoize to pull the return value from cache rather than running the filter plugin again. This can **drastically** increase performance assuming caching is acceptable per your use case.

Consider this scenario, an IDS alert has been received but the alert only contains the source_ip and destination_ip. However, having the DNS name associated with these IP addresses could be valuable to an analyst. If DNS queries and answers are in Elasticsearch, they can be pulled into the IDS alert automatically.

Here is the configuration to do it:
```javascript
filter {
    if [event_type] == "alert" {
        memoize {
            key => "%{destination_ip}"
            fields => [ "highest_registered_domain", "query" ]
            filter_name => "elasticsearch"
            filter_options => {
                query => "type:bro_dns AND answers:%{destination_ip}"
                index => "logstash-bro-*"
                fields => {
                    "query" => "destination_fqdn"
                    "highest_registered_domain" => "destination_highest_registered_domain"
                }
            }
        }
    }
}
```

This configuration would take the destination_ip and check to see if the Elasticsearch index logstash-bro-* had a previous answer that contained the destination_ip. If a match was found the elasticsearch filter plugin would pull back the highest_registered_domain and query fields. In this example, the query field would be stored into a field called destination_fqdn and the highest_registered_domain field would be stored into a field called destination_highest_registered_domain. Memoize would then cache this so that if the same alert came in 50 times the first alert would be cached and the remaining 49 would pull from that chace.

####Other use cases for the memoize filter plugin:

- Querying threat intelligence feeds stored in Elasticsearch indexes (**Collective Intelligence Framework** uses Elasticsearch)
- Building your own custom intelligence feeds and querying them
- Querying custom whitelist information (in house controlled whitelisting feeds can be extremely powerful)

There are other ways to do the above but this is another method often overlooked.

---------
### geoip
The **geoip** plugin is a filter plugin used to take an IP address and resolve it to geographic information such as city, state, latitude, longitude, and Autonomous System Number (ASN).

Here is an example of using geoip against a field called destination_ip:
```javascript
filter {
    geoip {
      source => "[destination_ip]"
    }
}
```

Here is an example of using geoip against a field called destination_ip and saving the results to a field called destination_geo:
```javascript
filter {
    geoip {
      source => "[destination_ip]"
      target => "destination_geo"
    }
}
```

Here is an example of using geoip against a field called destination_ip using a custom geoip database (such as commercial use of MaxMind):
```javascript
filter {
    geoip {
      database => "/usr/local/share/GeoIP/GeoLiteCity.dat"
      source => "[destination_ip]"
      target => "destination_geo"
    }
}
```

The above examples only retrieve traditional geo information such as city, state, latitude, and longitude. It does not include ASN which is one of the most underutilized and tactically important fields.

The below information shows the standard geoip information for 8.8.8.8:

```bash
destination_ip                      8.8.8.8
destination_geo.area_code 		    650
destination_geo.city_name 		    Mountain View
destination_geo.continent_code 	    NA
destination_geo.country_code2 		US
destination_geo.country_code3 		USA
destination_geo.country_name 		United States
destination_geo.ip 		            8.8.8.8
destination_geo.latitude 		    37.386
destination_geo.location 		    -122.084, 37.386
destination_geo.longitude 		    -122.084
destination_geo.postal_code 		94035
destination_geo.real_region_name 	California
destination_geo.region_name 		CA
destination_geo.timezone 		    America/Los_Angeles
```

While this is helpful, it is does not add enough context for the analyst. However, the ASN does. See the ASN information for 8.8.8.8:

```bash
destination_ip                      8.8.8.8
destination_geo.asn 		        Google Inc.
destination_geo.number 		        AS15169
```

Without using DNS, this ASN of 15169 shows that 8.8.8.8 is registered to Google Inc. This is powerful for filtering and applying additional Logstash configurations. For example, you could use the ASN to filter out certain businesses such as Microsoft from specific hunting techniques.

Below is the configuration used to pull in ASN information:

```javascript
filter {
    geoip {
        database => "/usr/local/share/GeoIP/GeoIPASNum.dat"
        source => "[source_ip]"
        target => "source_geo"
    }
}
```

It simply requires pointing at a ASN database file.

---------
### mutate
The **mutate** plugin is a filter plugin used to alter a log. It has many purposes as seen below.

#### Add a field
This example config adds a field called hostname_resolved and starts it with an empty value:
```javascript
filter {
    mutate {
        add_field => { "hostname_resolved" => "" }
    }
}
```

This example config adds a field called hostname_resolved and clones the value of the hostname field into it:
```javascript
filter {
    mutate {
        add_field => { "hostname_resolved" => "%{hostname}" }
    }
}
```

#### Add a tag
This example config adds a tag called pci:
```javascript
filter {
    mutate {
        add_tag => "pci"
    }
}
```

This example config adds a tag called pci and critical_asset:
```javascript
filter {
    mutate {
        add_tag => [ "pci", "critical_asset" ]
    }
}
```

#### Convert a field type (integer, float, string, boolean)
This example config converts the field source_port to a integer:
```javascript
filter {
    mutate {
        convert => [ "source_port", "integer" ]
    }
}
```

#### String Replacement (gsub)
This example config replaces "= " with "=na " in the syslog_message field:
```javascript
filter {
    mutate {
        gsub => [ "syslog_message", "= ", "=na " ]
    }
}
```

#### Add unique ID
This example config adds a unique ID to a log into a field called unique_id:
```javascript
filter {
    mutate {
        id => "unique_id"
    }
}
```

#### lowercase all text
This example config lowercases all letters in the syslog_message field:
```javascript
filter {
    mutate {
        lowercase => [ "syslog_message" ]
    }
}
```

#### Remove unwanted/redundant field(s)
This example config removes the syslog_message field:
```javascript
filter {
    mutate {
        remove_field => [ "syslog_message" ]
    }
}
```

#### Remove tag(s)
This example config removes the alert tag:
```javascript
filter {
    mutate {
        remove_tag => [ "alert" ]
    }
}
```

#### Rename / Standardize field name(s)
This example config renames the src_ip field to source_ip:
```javascript
filter {
    mutate {
        rename => [ "src_ip", "source_ip" ]
    }
}
```

**Standardizing field names is CRITICAL**

#### Replace a field's value
This example config replaces the value of host to 172.16.0.2:
```javascript
filter {
    mutate {
        replace => { "host" => "172.16.0.2" }
    }
}
```

#### Remove whitespace from beginning and end of field
This example config removes whitespace from the beginning and end of the syslog_message field:
```javascript
filter {
    mutate {
        strip => [ "syslog_message" ]
    }
}
```

#### uppercase all text
This example config uppercase all letters in the syslog_message field:
```javascript
filter {
    mutate {
        uppercase => [ "syslog_message" ]
    }
}
```

---------
### rest
The **rest** plugin is a community filter plugin used to submit a web based REST request. It is used to query APIs or websites using data from a log. This plugin can be very powerful when used properly.

#### Recommended use cases:

- Entropy (randomness checking) of key fields in conjunction with freq_server.py
- DNS top 1 million checking in conjunction with domain_stats.py
- WHOIS creation date lookups in conjunction with domain_stats.py

This example configuration is used get the entropy score of a DNS domain using the highest_registered_domain field's value (requires freq_server.py listening on 10004):
```javascript
filter {
    rest {
        request => {
            url => "http://localhost:20001/domain/creation_date/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "domain_creation_date"
    }
}
```

Another example:

```javascript
filter {
    rest {
        request => {
            url => "http://localhost:10004/measure/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "domain_frequency_score"
    }
    if [domain_frequency_score] {
        mutate {
            convert => [ "domain_frequency_score", "float" ]
        }
    }
}
```

This example configuration is used to find out the creation date of the highest_registered_domain value (requires domain_stats.py listening on 20000):
```javascript
filter {
    rest {
        request => {
            url => "http://localhost:20000/alexa/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "site"
    }
    if [site] != "0" and [site] {
        mutate {
            add_tag => [ "top-1m" ]
            remove_field => [ "site" ]
        }
    }
}
```

This example configuration is used to find out if the highest_registered_domain value is a top 1 million site (requires domain_stats.py listening on 20000):
```javascript
filter {
    rest {
        request => {
            url => "http://localhost:20000/alexa/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "site"
    }
    if [site] != "0" and [site] {
        mutate {
            add_tag => [ "top-1m" ]
            remove_field => [ "site" ]
        }
    }
}
```

---------
### ruby
The **ruby** plugin is a filter plugin that allows raw programming logic. It allows invoking ruby to programmatically interact with fields and field data.

#### Recommended use cases:

- Calculate field length
- Perform mathmatic calculations
- Calculate how long Logstash takes to do something

This example configuration is used to calculate the field length of a field called certificate_common_name and to store it in a field called certificate_common_name_length:
```javascript
filter {
    ruby {
        code => "event['certificate_common_name_length'] = event['certificate_common_name'].length;"
    }
}
```

This example configuration is used to calculate the number of days a certificate is valid by substracting the certificate_not_valid_after field from the certificate_not_valid_before field and then rounding the valid to an integer:
```javascript
filter {
    ruby {
        code => "event['certificate_number_days_valid'] = ((event['certificate_not_valid_after'] - event['certificate_not_valid_before']) / 86400).ceil;"
    }
}
```

This example decodes a base64 value stored in a field called possible_base64_code:
```javascript
filter {
    ruby {
        init => "require 'base64'"
        code => "a = Base64.decode64(event['possible_base64_code']);
                 event['base64_decoded'] = a;"
    }
}
```

This example extracts every instance of PowerShell cmdlets being used in EventID 4103 by using scan:
```javascript
filter {
    if [Payload] and [EventID] == 4103 and [SourceName] == "Microsoft-Windows-PowerShell" {
        ruby {
            code => "event['cmdlets'] = event['Payload'].downcase.scan(/commandinvocation\(([a-z0-9-]+)\)/)"
        }
    }
}
```


This example configuration is used to calculate how long it takes Logstash to process something:
```javascript
filter {
    ruby {
        code => "event['task_start'] = Time.now.to_f;"
    }
    Then do something...
    ruby {
        code => "event['task_end'] = Time.now.to_f;"
    }
    ruby {
        code => "event['logstash_time'] = event['task_end'] - event['task_start']"
    }
    mutate {
        remove_field => [ 'task_start', 'task_end' ]
    }
}
```

The overhead of calculating logstash_time is nominal. This likely can be left on.


This example configuration is used to calculate the ratio of bytes uploaded vs bytes downloaded using flow data:
```javascript
filter {
    ruby {
        code => "event['byte_ratio_client'] = event['bytes_to_client'].to_f / event['bytes_to_server'].to_f"
    }
    ruby {
        code => "event['byte_ratio_server'] = 1 - event['byte_ratio_client']"
    }
}
```
    
This example configuration is used to take an IDS alerts SID # and use it to retrieve the IDS rule it belongs to and append that rule to the alert:
```javascript
filter {
    if [gid] == 1 and [sid] {
        ruby {
            code => "sid = event['sid']; event['rule'] = `cat /etc/nsm/rules/*.rules | grep sid:#{sid} | head -n1`;"
        }
    }
}
```

---------
### syslog_pri
The **syslog_pri** plugin is a filter plugin used to automatically parse the syslog pri field into severity and priority fields.

This is an example configuration using the default message field:
```javascript
filter {
    syslog_pri { }
}
```

This is an example configuration using a field called syslog_pri:
```javascript
filter {
    syslog_pri { 
        source => "syslog_pri"
    }
}
```

---------
### tld
The **tld** plugin is a filter plugin used to take a DNS name and break it up into corresponding pieces. For example, www.google.com would become:

**highest_registered_domain** = google.com
**sub_domain** = www
**parent_domain** = google
**top_level_domain** == com

This is an example configuration of using tld against a field called query:
```javascript
filter {
    tld {
        source => "query"
    }
    mutate {
        rename => { "[tld][domain]" => "highest_registered_domain" }
        rename => { "[tld][trd]" => "sub_domain" }
        rename => { "[tld][tld]" => "top_level_domain" }
        rename => { "[tld][sld]" => "parent_domain" }
    }
}
```

---------
### translate
The **translate** plugin is a filter plugin used to take a field and look up a value based on it in a file or provided array of values.

This is an example configuration that takes the value of destination_port and does a lookup of its value in /lib/dictionaries/iana_services.yaml:
```javascript
filter {
    translate {
        field => "[destination_port]"
        destination => "[destination_service]"
        dictionary_path => "/lib/dictionaries/iana_services.yaml"
    }
}
```

This could help take a destination_port of 80 and use it to add a field called destination_service with a value of HTTP.


Output Plugins
---------
- [stdout](#stdout)
- [elasticsearch](#output_elasticsearch)
- [file](#output_file)
- [rabbitmq](#output_rabbitmq)
- [kafka](#output_kafka)
- [tcp](#output_tcp)
- [udp](#output_udp)

---------
### stdout
The **stdout** plug is an output plugin used to output to the screen. It is useful for troubleshooting or testing.

This example configuration is used to output to the screen with pretty markup. This is the most common way to invoke **stdout** and is probably what you want to use.
```javascript
output {
  stdout { codec => rubydebug }
}
```

Using the above configuration will display output similar to this:
```bash
{
             "message" => "2017-07-25T17:49:37.356Z sec-555-linux 1496523628.328546\tCfnUMi23OlkVdjx0Wi\t10.0.1.11\t38938\t10.0.0.10\t53\tudp\t46693\t0.001092\tlogingest.test.int\t1\tC_INTERNET1\tA\t0\tNOERROR\tT\tF\tTT0\t172.16.1.10\t3600.000000\tF",
            "@version" => "1",
          "@timestamp" => "2017-07-25T17:50:10.710Z",
                "host" => "sec-555-linux",
           "timestamp" => "2017-07-25T17:49:37.356Z sec-555-linux 1496523628.328546",
                 "uid" => "CfnUMi23OlkVdjx0Wi",
           "source_ip" => "10.0.1.11",
         "source_port" => "38938",
      "destination_ip" => "10.0.0.10",
    "destination_port" => "53",
            "protocol" => "udp",
      "transaction_id" => "46693",
                 "rtt" => "0.001092",
               "query" => "logingest.test.int",
         "query_class" => "1",
    "query_class_name" => "C_INTERNET1",
          "query_type" => "A",
     "query_type_name" => "0",
               "rcode" => "NOERROR",
          "rcode_name" => "T",
                  "aa" => "F",
                  "tc" => "TT0",
                  "rd" => "172.16.1.10",
                  "ra" => "3600.000000",
                   "z" => "F"
}
```

The alternative way to run **stdout** is to just invoke it such as in this configuration:
```javascript
output {
  stdout { }
}
```

However, the output is not user friendly and will look like this:

```bash
2017-07-25T17:49:37.356Z sec-555-linux 1496523628.328546	CfnUMi23OlkVdjx0Wi	10.0.1.11	38938	10.0.0.10	53	udp	46693	0.001092	logingest.test.int	1	C_INTERNET1	A	0	NOERROR	T	F	TT0	172.16.1.10	3600.000000	F
```

### output_elasticsearch
The **elasticsearch** plugin is an output plugin used to send logs to an Elasticsearch index.

This example configuration is used to send logs to an index for Windows logs:
```javascript
output {
    elasticsearch {
        index => "logstash-windows-%{+YYYY.MM.dd}"
    }
}
```

This example configuration shows dynamically routing logs to varies Elasticsearch indexes:
```javascript
output {
    if [type] == "windows" {
        elasticsearch {
            index => "logstash-windows-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "alert" {
        elasticsearch {
            index => "logstash-alert"
        }
    }
    if "syslog" == [tags] {
        elasticsearch {
            index => "logstash-syslog-%{+YYYY.MM.dd}"
        }
    }
}
```

---------
### output_file
The **file** output plugin is an output plugin used to send logs to a file.

This example configuration is used to send logs to a file called /home/student/logs.json:
```javascript
output {
    file {
        path => "/home/student/logs.json"
    }
}
```

It is possible to output logs to other file formats such as CSV. For CSV, use the **csv** output plugin.

---------
### output_rabbitmq
The **rabbitmq** output plugin is an output plugin used to send logs to RabbitMQ which is a third party message broker/log buffer.

This example configuration is used to send logs to an exchange for Windows logs:
```javascript
output {
    rabbitmq {
        key => "routing_key_goes_here"
        exchange => "windows"
        exchange_type => "direct"
        user => "user_name_goes_here"
        password => "password_goes_here"
        host => "rabbitmq_hostname_goes_here"
        port => 5672
        durable => true
        persistent => true
    }
}
```

It is a good practice to route logs to various queues so that you can monitor and troubleshoot them individually. This is an example of doing so:

```javascript
output {
    if [type] == "windows" {
        rabbitmq {
            key => "routing_key_goes_here"
            exchange => "windows"
            exchange_type => "direct"
            user => "user_name_goes_here"
            password => "password_goes_here"
            host => "rabbitmq_hostname_goes_here"
            port => 5672
            durable => true
            persistent => true
        }
    }
    if "syslog" in [tags] {
        rabbitmq {
            key => "routing_key_goes_here"
            exchange => "syslog"
            exchange_type => "direct"
            user => "user_name_goes_here"
            password => "password_goes_here"
            host => "rabbitmq_hostname_goes_here"
            port => 5672
            durable => true
            persistent => true
        }
    }
}
```

---------
### output_kafka
The **kafka** output plugin is an output plugin used to send logs to Kafka which is a third party message broker/log buffer.

This example configuration is used to send logs to a topic for Windows logs:
```javascript
output {
    kafka {
        bootstrap_servers => "kafka_server_name_goes_here:9092"
        topic_id => "syslog"
    }
}
```

It is a good practice to route logs to various topics so that you can monitor and troubleshoot them individually. This is an example of doing so:

```javascript
output {
    if [type] == "windows" {
        kafka {
            bootstrap_servers => "kafka_server_name_goes_here:9092"
            topic_id => "windows"
        }
    }
    if "syslog" in [tags] {
        kafka {
            bootstrap_servers => "kafka_server_name_goes_here:9092"
            topic_id => "syslog"
        }
    }
}
```

---------
### output_tcp
The **tcp** output plugin is an output plugin used to send logs to a remote logging host. It can be used to send logs from Logstash to a commercial SIEM.

This example configuration is used to send logs to a remote host over TCP port 5000
```javascript
output {
    tcp {
        host => "dns_name_or_ip_address_goes_here"
        port => 5000
    }
}
```

This example configuration is used to send logs with a type of **alert** to a remote host over TCP port 5000
```javascript
output {
    if [type] == "alert" {
        tcp {
            host => "dns_name_or_ip_address_goes_here"
            port => 5000
        }
    }
}
```

---------
### output_udp
The **udp** output plugin is an output plugin used to send logs to a remote logging host. It can be used to send logs from Logstash to a commercial SIEM.

This example configuration is used to send logs to a remote host over UDP port 5000
```javascript
output {
    udp {
        host => "dns_name_or_ip_address_goes_here"
        port => 5000
    }
}
```

This example configuration is used to send logs with a type of **alert** to a remote host over UDP port 5000
```javascript
output {
    if [type] == "alert" {
        udp {
            host => "dns_name_or_ip_address_goes_here"
            port => 5000
        }
    }
}
```

Additional Info
--------------
