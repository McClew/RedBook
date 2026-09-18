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

# 49 - TACACS+

**TACACS+** (Terminal Access Controller Access-Control System Plus) is a Cisco-developed AAA (Authentication, Authorization, Accounting) protocol used mainly to centralise login and command-authorisation for network devices — routers, switches, and firewalls authenticate administrators against a TACACS+ server instead of local accounts.

{% hint style="info" %}
TACACS+ listens on **TCP port 49** (occasionally seen over UDP in older/non-standard deployments). Unlike its cousin RADIUS, TACACS+ encrypts the **entire packet body**, not just the password field, using a pre-shared key configured on both the client (the network device) and the server.
{% endhint %}

```
PORT   STATE SERVICE
49/tcp open  tacacs
```

***

## Enumeration

| Command                     | Description                                                                                                             |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `nmap -sV -p49 <TARGET_IP>` | Confirms the service and often reveals the TACACS+ daemon in use (e.g. Cisco ACS, `tac_plus`, ClearPass).               |
| `nc -nv <TARGET_IP> 49`     | The service will not send a useful banner — packets are binary and encrypted, so this mainly confirms the port is open. |

Because every field beyond the header is encrypted with the shared key, **there is no unauthenticated information disclosure** in a normal TACACS+ exchange the way there is with, say, SNMP or FTP banners.

***

## Attacking the Shared Secret

* **Capture and offline-crack**: if you can position yourself on the path between a network device and the TACACS+ server (e.g. via ARP spoofing on an internal engagement), capture the exchange with Wireshark (which has a native TACACS+ dissector) and attempt an offline brute-force of the shared key using scripts built on Python's `tacacs_plus` library.
* **Configuration exposure**: the shared key is stored in plaintext or a reversible encoding in device configs (`tacacs-server key <KEY>` on Cisco IOS). A configuration backup, TFTP capture, or `show run` obtained through another foothold is usually a far faster route to the key than cracking captured traffic.
* Once the shared key is known, it can be used to decrypt captured sessions (recovering admin credentials) or to stand up a rogue TACACS+ server that network devices will trust.

{% hint style="warning" %}
Misconfigured devices sometimes fall back to **local authentication** if the TACACS+ server is unreachable. Forcing a denial of service against the TACACS+ server (or blocking port 49) can, on a badly configured device, cause it to accept a known local/default account instead.
{% endhint %}
