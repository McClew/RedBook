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

# Masscan

## Cheat Sheet

<table><thead><tr><th width="388.333251953125">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>masscan &#x3C;ip_address|network_prefix> --ports &#x3C;80></code></td><td>Scan an IP or network subnet for port 80</td></tr><tr><td><code>masscan &#x3C;10.0.0.0/16> --top-ports &#x3C;100> --rate &#x3C;100000></code> </td><td>Scan a class B subnet for the top 100 ports at 100,000 packets per second</td></tr><tr><td><code>masscan &#x3C;10.0.0.0/16> --top-ports &#x3C;100> --excludefile &#x3C;path/to/file></code></td><td>Scan a class B subnet avoiding ranges from a specific exclude file</td></tr><tr><td><code>masscan &#x3C;0.0.0.0/0> --ports &#x3C;443> --rate &#x3C;10000000></code></td><td>Scan the Internet for port 443</td></tr><tr><td><code>masscan &#x3C;0.0.0.0/0> --ports &#x3C;0-65535> -output-format &#x3C;binary|grepable|json|list|xml> --output-filename &#x3C;path/to/file></code></td><td>Scan the Internet for a specific port range and export to a file</td></tr></tbody></table>
