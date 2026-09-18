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

# 43 - WHOIS

**WHOIS** is a simple, unauthenticated, plain-text query/response protocol used to look up registration records for domain names, IP address blocks, and autonomous systems held by registries and registrars.

{% hint style="info" %}
WHOIS servers listen on **TCP port 43**. A query is a single line terminated with `\r\n`; the server replies with free-text data and then closes the connection — there is no framing or authentication.
{% endhint %}

```
PORT   STATE SERVICE
43/tcp open  whois
```

***

## Querying a WHOIS Server

{% code title="Using the whois client" %}
```bash
whois -h <TARGET_IP> "example.com"
```
{% endcode %}

{% code title="Raw protocol (no client installed)" %}
```bash
echo -e "example.com\r" | nc <TARGET_IP> 43
```
{% endcode %}

If no server is specified, `whois` resolves the correct authoritative registry automatically based on the TLD or IP block.

***

## What to Pull From the Response

* Registrant / registrar / admin / tech contact details (names, emails, phone numbers) — useful for **OSINT and phishing pretexting**.
* Name servers and creation/expiry dates — useful for **domain takeover** checks (expired domains still referenced by DNS or SaaS integrations).
* For IP WHOIS (RIPE/ARIN/APNIC/LACNIC/AFRINIC): the owning organisation and netblock, which helps confirm whether an IP is in scope and map an organisation's full external footprint.

{% hint style="warning" %}
When a target organisation runs its **own internal WHOIS server** (rare, but seen on some enterprise IPAM/registry deployments), the same protocol can leak internal asset ownership, hostnames, and contact details never meant to be public. Treat an open port 43 on an internal engagement as worth querying, not just skipped past as "just WHOIS".
{% endhint %}

***

## Automated

```bash
nmap -sV -p43 --script whois-domain,whois-ip <TARGET_IP>
```

See also the dedicated **WHOIS** page under Web Enumeration for using this protocol as part of external domain recon against a target organisation.
