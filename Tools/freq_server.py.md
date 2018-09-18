freq_server.py
========

Abstract
--------

freq_server.py is a python API designed by Mark Baggett to handle mass entropy testing. It was designed to be used in conjunction with a SIEM solutions but can work with anything that can submit a web request.

**Background:** adversaries attempt to bypass signature based/pattern matching/blacklist techniques by introducing random: filenames, service names, workstation names, domains, hostnames, SSL cert subjects and issuer subjects, etc.

**Problem:** detecting randomly-generated X is a powerful defensive technique, but hard with a narrow scope.

**Solution:** ask Mark Baggett for help ...freq_server.py is born

freq_server.py helps us solve the problem by employing frequency tables that map how likely one character will follow another

Where to Acquire
---------

https://github.com/MarkBaggett/MarkBaggett/blob/master/freq/freq_server.py


Examples/Use Case
---------

### Using Logstash to query freq_server.py

This example Logstash configuration below queries freq_server.py for the entropy score of a domain name (stored in a field called highest_registered_domain). The returned entropy score is then saved into a field called domain_frequency_score.
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
}
```
Note: The values returned by the rest filter plugin will be strings. If you want them to be floats add this code below the rest filter:
```javascript
mutate {
    convert => [ "domain_frequency_score", "float" ]
}
```

The below command is an example of running freq_server.py on port 10004 and using a frequency table of /opt/freq/dns.freq. It does not require root or admin privileges.
```bash
/usr/bin/python /opt/freq/freq_server.py 10004 /opt/freq/dns.freq
```

This is an example of manually querying freq_server.py using curl. It requests the entropy score of sec555.com.
```bash
$ curl http://127.0.0.1:10004/measure/sec555.com
20.4191174097
```

To generate a custom frequency table see [freq.py](/Tools/freq.py.md). To view additional command line parameters see either the GitHub link above or run the following command:

```bash
/usr/bin/python /opt/freq/freq_server.py -h
```

---

Additional Info
--------------

https://isc.sans.edu/forums/diary/Continuous+Monitoring+for+Random+Strings/20451/