The Bro Network Security Monitor
========

Abstract
--------

> *Bro is an open-source network security platform that illuminates your network's activity in detail, with the stability and flexibility for production deployment at scale.*

> *Bro reduces incoming packet streams into higher-level events and applies customizable scripts to determine the necessary course of action. This simple design allows you to configure an array of real-time alerts, execute arbitrary programs on demand, and log data for later use.*

Source: <a href='https://www.bro.org/why_choose_bro.pdf' target='_blank'>Why Choose Bro</a> 


Where to Acquire
---------
Installed in the Security511 Linux VM.

Web site is: <a href='https://www.bro.org' target='_blank'>www.bro.org</a>

Examples/Use Case
---------

**Note:** Some of the examples below presume files and paths that might not match your particular system and tool installation.

Run bro against a pcap, create bro log files in the current directory. Some of the following logs files may be created, dpending on the pcap content and bro configuration:

* conn.log
* dns.log
* files.log
* http.log
* irc.log
* packet_filter.log
* ssl.log
* weird.log


```bash
$ bro -r /pcaps/virut-worm.pcap
```

Carve executables from a file:
```bash
$ sudo bro -r /pcaps/virut-worm.pcap /opt/bro/share/bro/file-extraction/extract.bro
$ ls -la /nsm/bro/extracted
```

Carve multiple file types: exe, txt, jpg, png, html and "other" (uses the extension .xxx):
```bash
$ sudo bro -r /pcaps/virut-worm.pcap /opt/bro/share/bro/file-extraction/extract-all.bro
$ ls -la /nsm/bro/extracted
```

Display x.509 issuer subjects:
```bash
$ bro -C -r /pcaps/normal/https/alexa-top-500.pcap
$ cat ssl.log | bro-cut issuer_subject
```

View Bro logs without wrapping lines
```bash
$ bro -r /pcaps/tbot.pcap
$ less -S http.log 
```

**Note**: Using `less -s` against Bro logs can be exremely useful before you know which fields to pull with `bro-cut`

Additional Info
--------------
A printable PDF version of this cheatsheet is available here:
[Bro](pdfs/Bro.pdf)

Cheat Sheet Version
--------------
#### **`Version 1.0`**
