---
icon: file-certificate
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Certificate Transparency

`Certificate Transparency` (`CT`) logs contain all certificates issued by a participating `Certificate Authority` (`CA`) for a specific domain. Therefore, `SSL/TLS certificates` from web servers include `domain names`, `subdomain names`, and `email addresses`.

Since these logs are public and accessible to everyone, it is a valuable source for understanding the target company's infrastructure better.&#x20;

We can use a tool that outputs all the `CT logs` for a target domain from different sources and filtered is <a href="https://github.com/UnaPibaGeek/ctfr" target="_blank">ctfr.py</a>.

***

## ctfr.py

{% code title="" overflow="wrap" lineNumbers="true" %}
```bash
McClew@htb[/htb]$ python3 ctfr.py -d inlanefreight.com

          ____ _____ _____ ____  
         / ___|_   _|  ___|  _ \ 
        | |     | | | |_  | |_) |
        | |___  | | |  _| |  _ < 
         \____| |_| |_|   |_| \_\
    
    Made by Sheila A. Berta (UnaPibaGeek)
    
inlanefreight.com
vc.inlanefreight.com
wlan.inlanefreight.com
afdc0102.inlanefreight.com
autodiscover.inlanefreight.com
kfdcex07.inlanefreight.com
kfdcex08.inlanefreight.com
videoconf.inlanefreight.com
<SNIP>
```
{% endcode %}
