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

# 111 - Portmapper

**Portmapper** (also called **RPCbind** in its modern implementation) is the directory service for **ONC RPC** (Open Network Computing Remote Procedure Call). Rather than RPC services listening on fixed, well-known ports, each one registers itself with the portmapper at start-up; a client first asks the portmapper which port a given RPC program is currently using, then connects to that port directly.

{% hint style="info" %}
Portmapper listens on **TCP and UDP port 111**. It is most commonly found on Unix/Linux hosts running **NFS**, and its presence is a strong hint that NFS shares, NIS, or other RPC-based services are also exposed.&#x20;
{% endhint %}

```
PORT    STATE SERVICE
111/tcp open  rpcbind
111/udp open  rpcbind
```

***

## Enumeration

{% code title="List registered RPC programs" %}
```bash
rpcinfo -p <TARGET_IP>
```
{% endcode %}

Typical output maps a **program number** to the **port** it is currently listening on:

```
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100003    3   tcp   2049  nfs
    100005    3   tcp  20048  mountd
    100021    4   tcp  36849  nlockmgr
```

From here, cross-reference well-known program numbers (100003 = NFS, 100005 = mountd, 100024 = status, 100021 = nlockmgr, 100009 = yppasswdd, 100004 = ypserv/NIS) to decide what to enumerate next — typically NFS export lists with `showmount -e <TARGET_IP>`.

***

## Automated

```bash
nmap -sV -p111 --script rpcinfo <TARGET_IP>
sudo nmap -sSU -p111 --script=rpcinfo <TARGET_IP>   # also check UDP
```

***

### Notes

* A Portmapper response with no NFS/NIS behind it is still useful — the registered program list is a map of every RPC-based service on the box, some of which (old NIS, rpc.rusersd, rpc.rquotad) leak usernames or system info with no authentication at all.
* Historically, Portmapper itself has been abused for **UDP amplification** and, on misconfigured firewalls, to reach RPC services that were intended to be internal-only by bouncing requests through the portmapper's forwarding behaviour.
* See also: **RPCBind**, for daemon-specific notes on the modern `rpcbind` implementation that provides this service on current Linux distributions.
