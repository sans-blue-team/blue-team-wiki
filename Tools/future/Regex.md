Regex (Regular Expressions)
========

Abstract
--------

Monitoring and analysis routinely involves pattern matching. Regular expressions reason for existence is to facilitate pattern matching. Regex can seem like a language on its own, especially when initially encountering it.

Where to Acquire
---------
We don't typically need to acquire anything for regex as tools make use of regular expressions as a built-in feature. We just need to know how to write them in the way the tool expects.

**Note**: One frustrating thing about regular expressions is that there are different variants or flavors of regex. PCRE, Perl Compatible Regular Expressions, and POSIX are two common flavors that we will regularly encounter with standard tools.

Examples/Use Case
---------

<table><tr bgcolor="grey">
<th align="left">ModSecurity Rule Components<th align="left">Description</tr>
<tr><th>**`SecRule`**</th><td>The most basic ModSecurity directive that creates a rule.</td></tr>
<tr><th>**`RESPONSE_BODY`**</th><td>The part of HTTP the rule acts upon.</td></tr>
<tr><th>**`"EXFILEXFIL"`**</th><td>This is the content to be matched, which is where we supply the HoneyToken value.</td></tr>
<tr><th>**`phase:4`**</th><td>This is the phase of ModSecurity processing where the data will be accessible. Phase 4 is the Response Body phase.</td></tr>
<tr><th>**`msg:'Pilots HoneyToken Exfil Detected'`**</th><td>msg, allows configuration of a custom message "Pilots HoneyToken Exfil Detected" that will be associated with this particular rule.</td></tr>
<tr><th>**`tag:'HONEYTOKEN EXFILTRATION'`**</th><td>The tag action applies a tag that is used to categorize data. For example, there could be other rules that all fall under the same tag of, HONEYTOKEN EXFILTRATION.</td></tr></table>

Additional Info
--------------

[http://www.regular-expressions.info/](http://www.regular-expressions.info/)

Cheat Sheet Version
--------------
#### **`Version 1.0`**
