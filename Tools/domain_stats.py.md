domain_stats.py
========

Abstract
--------

domain_stats.py is a python API designed by Mark Baggett to handle Alexa/Cisco Umbrella top one million lookups as well as WHOIS lookups, both of which cache results. It was designed to be used in conjunction with a SIEM solutions but can work with anything that can submit a web request.

**Background:** adversaries attempt to often use domain names to bypass IP address blacklisting technologies. These could be in the form of random domain names (see freq.py and freq_server.py for detecting these), rapidly rotating domain names, or targeted domains such as phishing domains

**Problem:** analysts struggle to figure out what data to analyze and which techniques to apply against which logs. There are just too many logs...

**Solution:** ask Mark Baggett for help ...domain_stats.py is born

domain_stats.py helps us solve the problem by providing a lookup table using the Alexa or Cisco Umbrella top-1m.csv (can be pointed to a custom file as well) and WHOIS lookups to pull back information such as a domain's creation date.

Where to Acquire
---------

https://github.com/MarkBaggett/domain_stats


Examples/Use Case
---------

### Using Logstash to query domain_stats.py against top-1m

This example Logstash configuration below queries domain_stats.py to see if a domain name (stored in a field called highest_registered_domain) is a member of the Alexa/Cisco Umbrella top 1 million sites. If the site is a top 1 million site a tag of "top-1m" is added to the log.
```javascript
filter {
    rest {
        request => {
            url => "http://localhost:20000/alexa/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "site_rank"
    }
    if [site_rank] != "0" and [site_rank] {
        mutate {
            add_tag => [ "top-1m" ]
        }
    }
}
```

Note: The values returned by the rest filter plugin will be strings. If you want them to be integers add this code below the rest filter:
```javascript
mutate {
    convert => [ "site_rank", "integer" ]
}
```

This example Logstash configuration below queries domain_stats.py to see when a domain name was created. It stores the results in the creation_date field.
```javascript
filter {
    rest {
        request => {
            url => "http://localhost:20000/domain/creation_date/%{highest_registered_domain}"
        }
        sprintf => true
        json => false
        target => "creation_date"
    }
}
```

The below command is an example of running domain_stats.py on port 20000 and using a top one million file at /opt/domain_stats/top-1m.csv. It does not require root or admin privileges.
```bash
/usr/bin/python /opt/domain_stats/domain_stats.py --preload 0 -a /opt/domain_stats/top-1m.csv 20000
```

Preload is set to 0 which means do not try to load up all the WHOIS information for the top one million sites. The default behavior loads the first 1000 sites listed in top-1m.csv or whatever file is specified.

To view additional command line parameters see either the GitHub link above or run the following command:

```bash
/usr/bin/python /opt/domain_stats/domain_stats.py -h
```

This is an example of manually querying domain_stats.py using curl. It requests the creation date from WHOIS information for sec555.com.
```bash
$ curl http://127.0.0.1:20000/domain/creation_date/sec555.com
2016-09-08 03:21:24;
```

This is an example of manually querying domain_stats.py using curl. It checks to see if sans.org is a top 1 million site. Since SANS is obviously freaking awesome... it is a top 1 million site and the rank of 105910 is returned.
```bash
$ curl http://127.0.0.1:20000/alexa/sans.org
105910
```

This is an example of manually querying domain_stats.py using curl. It checks to see if covertc2.com is a top 1 million site. Since it is not a value of "0" is returned.
```bash
$ curl http://127.0.0.1:20000/alexa/convertc2.com
0
```

This is an example of manually querying domain_stats.py using curl. It requests the WHOIS information for sec555.com.
```bash
$ curl http://127.0.0.1:20000/domain/sec555.com
{
  "updated_date": "2016-09-08 00:00:00", 
  "status": [
    "clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited", 
    "clientRenewProhibited https://icann.org/epp#clientRenewProhibited", 
    "clientTransferProhibited https://icann.org/epp#clientTransferProhibited", 
    "clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited", 
    "clientTransferProhibited http://www.icann.org/epp#clientTransferProhibited", 
    "clientUpdateProhibited http://www.icann.org/epp#clientUpdateProhibited", 
    "clientRenewProhibited http://www.icann.org/epp#clientRenewProhibited", 
    "clientDeleteProhibited http://www.icann.org/epp#clientDeleteProhibited"
  ], 
  "alexa": "0", 
  "name": "Justin Henderson", 
  "dnssec": "unsigned", 
  "city": "Effingham", 
  "expiration_date": [
    "2018-09-08 00:00:00", 
    "2018-09-08 03:21:24"
  ], 
  "time": 1497203898.527102, 
  "zipcode": "62401", 
  "domain_name": [
    "SEC555.COM", 
    "sec555.com"
  ], 
  "country": "US", 
  "whois_server": "whois.godaddy.com", 
  "state": "Illinois", 
  "registrar": "GoDaddy.com, LLC", 
  "referral_url": "http://www.godaddy.com", 
  "address": "14526 E Millbrook Dr", 
  "name_servers": [
    "NS55.DOMAINCONTROL.COM", 
    "NS56.DOMAINCONTROL.COM"
  ], 
  "org": null, 
  "creation_date": [
    "2016-09-08 00:00:00", 
    "2016-09-08 03:21:24"
  ], 
  "emails": [
    "abuse@godaddy.com", 
    "Justindestiny@gmail.com"
  ]
}
```

---

Additional Info
--------------

https://isc.sans.edu/forums/diary/Continuous+Monitoring+for+Random+Strings/20451/