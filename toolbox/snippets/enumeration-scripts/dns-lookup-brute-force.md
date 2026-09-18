---
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

# DNS Lookup Brute Force

Automate the forward DNS lookup of common hostnames using the host command and a simple bash script.

```bash
for ip in $(cat list.txt); do host $ip.acme.com; done
```

To create a wordlist of common hostnames:

```bash
echo www > list.txt
echo ftp >> list.txt
echo mail >> list.txt
echo owa >> list.txt
echo proxy >> list.txt
echo router >> list.txt
echo api >> list.txt
```
