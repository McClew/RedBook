---
icon: ethernet
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

# 113 - Ident

The **Identification Protocol** (Ident, RFC 1413) answers a very narrow question: for a given pair of local/remote TCP ports on the queried host, which **local user** owns that connection? It was designed to help mail and IRC servers attribute abuse to a specific account, not as a security control.

{% hint style="info" %}
Ident listens on **TCP port 113**. A query must reference an **existing, currently-open TCP connection** on the target - you can't query it for arbitrary ports and expect a meaningful answer.
{% endhint %}

```
PORT    STATE SERVICE
113/tcp open  ident
```

***

## Querying

The query format is `<local_port>, <remote_port>\r\n` from the perspective of the queried host. In practice this means you first need an existing connection to the target (e.g. an open FTP or IRC session) so you know which local port to ask about:

```bash
# From the box you connected FROM, note the source port of an existing connection, e.g. with:
ss -tn

# Then query Ident on the target for that connection's ports:
echo -e "<TARGET_LOCAL_PORT>, <YOUR_REMOTE_PORT>\r" | nc <TARGET_IP> 113
```

A successful response looks like:

```
6193, 23 : USERID : UNIX : root
```

revealing that the account behind the connection on port 23 is `root`.

***

## Value During Enumeration

* **Username disclosure**: confirms which OS account (often a service account) owns a given daemon - e.g. finding that the FTP daemon runs as `root` rather than a dedicated low-privilege user is a meaningful finding on its own.
* Historically paired with **IRC** and old **FTP bounce** techniques, where Ident was used to fingerprint the account running a service before attempting authentication or an exploit tailored to that account's privilege level.

***

## Automated

```bash
nmap -sV -p113 --script auth-owners <TARGET_IP>
```
