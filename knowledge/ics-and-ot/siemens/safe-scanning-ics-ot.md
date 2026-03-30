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

# Safe Scanning (ICS/OT)

## Slow & Specific

Standard Nmap defaults (like `-T4`) send packets too fast for the specialised Ethernet chips in Scalance switches and S7-1500 communication modules. If the "buffer" fills up, the device may stop processing real-time control traffic to handle the scan, triggering a CPU STOP.

* Never use `-T4` or `-T5`.
* Always use `-T2` (Polite) or `-T3` (Normal).
* Avoid `-p-` (all ports). Only scan known industrial ports.

***

## Recommended Nmap Syntax

Use this "Safety First" command for initial discovery:

```bash
nmap -sS -Pn -n -T2 --max-parallelism 1 -p 80,443,102,161,502 <TARGET_IP>
```

**Why these flags?**

* `-sS`: Stealth SYN scan (less resource intensive than a full connect).
* `-Pn`: Skip host discovery (don't ping, just scan).
* `-n`: Disables DNS resolution (keeps traffic off the gateway).
* `-T2`: Polite timing.
* `--max-parallelism 1`: Send one packet at a time, ensures we aren't hitting the PLC with 10 concurrent requests.
* `-p 102,443...`: Specifically targets the S7 Protocol (102) and Web Management.

***

## Safe Enumeration Scripts (NSE)

Nmap has built-in scripts specifically for Siemens that are generally safe because they use the "read-only" portion of the protocol.

For S7-1500 (Protocol Info):

```bash
nmap --script s7-info.nse -p 102 <TARGET_IP>
```

This sends a "Read SZL" request. This is a standard diagnostic query that returns the Module Name, Serial Number, and Firmware Version.

For Scalance (SNMP Info):

{% code overflow="wrap" %}
```bash
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=communities.txt <TARGET_IP>
```
{% endcode %}

{% hint style="info" %}
#### Safety Warning

Only use a small, targeted `communities.txt`. High-speed brute-forcing of SNMP on a Scalance switch can cause its management CPU to spike to 100%.
{% endhint %}

***

## Indicators of "Fragility" (The "Stop" Signs)

If you see these behaviors during a scan, STOP immediately and notify the plant operator:

1. High Latency: Pings suddenly jump from 2ms to 200ms.
2. Missing Packets: The device stops responding to every second or third request.
3. Red LEDs: If a "SF" (System Fault) or "BF" (Bus Fault) red light appears on the physical PLC hardware.

***

## The "No-Scan" Zone

Avoid scanning these ports on Siemens hardware unless explicitly required:

* Port 445 (SMB): PLCs don't use SMB. Scanning it often triggers "Illegal Packet" logs in industrial firewalls.
* Port 3389 (RDP): Only relevant if you are hitting the Engineering Workstation (the PC), not the PLC itself.
* Port 0: Some older Siemens CPs will crash if they receive a packet on Port 0.
