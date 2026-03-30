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

# Scalance Switches

Siemens Scalance is a brand of industrial networking hardware (Layer 2 and Layer 3) specifically engineered for the Operational Technology (OT) environment. Unlike standard IT switches, Scalance devices are "ruggedised" to withstand extreme temperatures, electromagnetic interference (EMI), and constant vibration.

***

## Purpose

In an industrial hierarchy, Scalance switches act as the nervous system connecting the PLC to the sensors, motors, and HMI panels. They are the primary transport layer for **Profinet**, the industrial Ethernet standard used by Siemens.

***

## Quick Wins

### Default Credentials

`admin` / `admin`

Since firmware updates around 2022-2024, Scalance switches force a password change upon the first login. However, if a switch has been "Factory Reset" but never re-configured, we may be able to log in with `admin/admin` and set our own password, effectively taking ownership of that network segment.

## Port Mirroring

If we gain access to the Scalance Web Based Management (WBM):

We can configure any port to be a Monitor Port (SPAN). Mirror the traffic from a PLC port to the attacker hosts port. We can then capture the `S7CommPlus` traffic to look for credentials or analyse the industrial logic being sent to the machines.

### SNMP

Most Scalance switches ship with SNMP enabled for monitoring.

Default Communities: `public` (Read) and `private` (Write).

Use `snmpwalk` to pull the ARP Table and CAM Table. This allows us to map exactly which PLC is plugged into which port without ever touching the PLC itself.

**Credential Leak**_:_ In some firmware versions, the configuration (including obfuscated passwords) can be pulled via specific SNMP OIDs.

### LLDP

Over LLDP (Link Layer Discovery Protocol), Scalance switches broadcast their identity every 30 seconds.&#x20;

{% code overflow="wrap" %}
```bash
tcpdump -i eth0 'ether proto 0x88cc'
```
{% endcode %}

Use the above command to see the switch's IP address, management VLAN, and model number without sending a single packet.

### DCP

DCP (Discovery & Configuration Protocol): This is a Siemens-specific Layer 2 protocol.

We can send a "DCP Identify" multicast packet. Every Scalance switch on the segment will shout back its IP, MAC, and "Name of Station." It is an unauthenticated protocol.

***

## Key Characteristics

* **Profinet Optimised:** They prioritise industrial traffic over standard TCP/IP to ensure that a motor stop command arrives in milliseconds, even if the network is busy.
* **DCP (Discovery and Configuration Protocol):** This is a unique Layer 2 protocol. Because many industrial devices don't have screens or keyboards, Scalance switches use DCP to allow a central engineering station (TIA Portal) to find and assign IP addresses to new hardware on the wire.
* **Redundancy Protocols (MRP):** To prevent a single cable failure from stopping a factory line, these switches often use the Media Redundancy Protocol (MRP) to create "rings." If one link breaks, the switch re-routes traffic in under 50ms.
* **Form Factor:** They typically mount on DIN rails and often use 24V DC power instead of standard 110V/230V AC.

***

## Targeting

1. Lateral Movement: Gaining access to a Scalance switch often grants a "god-view" of all PLC-to-PLC communication.
2. Traffic Mirroring: They are the ideal place to set up a SPAN port to sniff S7CommPlus traffic for credential harvesting or logic analysis.
3. Denial of Service (DoS): Because they handle real-time traffic, even a minor flood or a misconfigured SNMP scan can cause a "Safety Timeout," which physically shuts down the production machinery.
