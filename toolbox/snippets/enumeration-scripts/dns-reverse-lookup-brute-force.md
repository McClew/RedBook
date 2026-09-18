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

# DNS Reverse Lookup Brute Force

If a domain has PTR Record configured, domain names might be found using a reverse lookup script:

```sh
for ip in $(seq 155 190); do host 50.7.67.$ip; done | grep -v "not found"
```
