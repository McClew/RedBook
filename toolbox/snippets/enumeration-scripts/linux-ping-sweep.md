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
---

# Linux Ping Sweep

```bash
for i in $(seq 254); do ping 172.16.8.$i -c1 -W1 & done | grep from
```
