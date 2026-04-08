---
icon: ethernet
---

# 111 - RPCBind

`rpcbind` is a server utility that maps Remote Procedure Call (RPC) program numbers to universal addresses (TCP/UDP port 111). It acts as a directory service, enabling clients to find the correct port for services like NFS. It must run before RPC-based services start and is crucial for network file sharing.

{% hint style="info" %}
#### Default Port

111 TCP & UDP
{% endhint %}

The `rpcbind` service should not be exposed externally, so if this is the case write up a Low finding for Unnecessary Exposed Services or similar.

This port can be probed to fingerprint the operating system or potentially gather information about available services. We can try to probe it with the `rpcinfo` command or [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention").

***

## Basic Enumeration

```bash
rpcinfo <TARGET_IP>
```

{% code title="Example of the full execution and response." %}
```bash
clue@host[~]$ rpcinfo 10.129.203.101

   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /run/rpcbind.sock      portmapper superuser
    100000    3    local     /run/rpcbind.sock      portmapper superuser
```
{% endcode %}
