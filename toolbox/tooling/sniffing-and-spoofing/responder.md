# Responder

Responder is an [llmnr-nbt-ns-poisoning.md](../../../field-manual/exploitation/man-in-the-middle/llmnr-nbt-ns-poisoning.md "mention") and [mdns-poisoning.md](../../../field-manual/exploitation/man-in-the-middle/mdns-poisoning.md "mention") poisoner, with built-in HTTP/SMB/MSSQL/FTP/LDAP rogue authentication server supporting NTLMv1/NTLMv2/LMv2, Extended Security NTLMSSP and Basic HTTP authentication.

It answers specific name resolution queries that a Windows machine sends out when it cannot find a resource via DNS. By "poisoning" these requests, Responder tricks the victim into authenticating against the attacker machine, allowing us to capture NTLM hashes.

{% hint style="info" %}
## Download & Install:

[https://github.com/lgandx/Responder](https://github.com/lgandx/Responder)
{% endhint %}

***

## Cheatsheet

{% tabs %}
{% tab title="Parameters" %}
<table><thead><tr><th width="183">Parameter</th><th>Description</th></tr></thead><tbody><tr><td><code>-I &#x3C;interface></code></td><td>Required. Specifies which network card to listen on (e.g., eth0).</td></tr><tr><td><code>-A</code></td><td>Passive discovery. Sees what’s happening without sending a single packet.</td></tr><tr><td><code>-d</code></td><td>Enables the rogue DHCP server to inject DNS settings (Aggressive).</td></tr><tr><td><code>-P</code></td><td>Starts the rogue WPAD server to capture browser credentials.</td></tr><tr><td><code>-r</code></td><td>Redirects failed SMB searches to your machine (Useful for older OS).</td></tr><tr><td><code>-f</code></td><td>Attempts to fingerprint the OS of the machine sending the request.</td></tr><tr><td><code>-v</code></td><td>Provides more detail on the incoming requests and poisoning logic.</td></tr></tbody></table>
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

***

## Analyse Mode

Instead of actively responding to requests and poisoning the network, in 'Analyse Mode' Responder simply logs every LLMNR, NBT-NS, and MDNS request it hears.

**Why use it?**

**Stealth:** It produces zero network noise. You can gauge the volume of traffic without alerting an IDS/EDR.

**Scoping:** It tells you which hostnames are being looked for most often.

**Safety:** It ensures you don't accidentally poison a sensitive server or a production-critical service before you’ve cleared it with the client.

***

## Walkthrough: Basic Hash Capture

{% stepper %}
{% step %}
### Identify the Interface

Run `ip` a or `ifconfig` to find our active network interface (usually `eth0` or `ens33`).
{% endstep %}

{% step %}
### Start in Analyse Mode

Run:

```bash
sudo responder -I eth0 -A
```

Wait 5–10 minutes to see if the network is "chatty."
{% endstep %}

{% step %}
### Launch Active Poisoning

Once we're ready to capture, run:

```bash
sudo responder -I eth0 -dwv
```

The `-dw` flags help catch WPAD and DHCP-based requests which are common "easy wins".
{% endstep %}

{% step %}
### Monitor for Hashes

The screen will light up with \[SMB] NTLMv2-SSP Hash when a user attempts to access a non-existent network share.
{% endstep %}

{% step %}
### Collect & Crack

The hashes are automatically saved in `/usr/share/responder/logs/`. We can then move these to a cracking station for use with tools like [hashcat.md](../password-attacks/hashcat.md "mention").
{% endstep %}
{% endstepper %}

***

## Limitations & Risks

### Network Congestion

In very large flat networks, Responder can generate a massive amount of traffic.

### The "Account Lockout" Trap

If you use the captured hashes in an automated tool (like [crackmapexec.md](../post-exploitation/crackmapexec.md "mention")) against the whole domain, you might trigger account lockout policies.

### Service Disruption

Poisoning WPAD can occasionally cause browser lag or connectivity issues for end-users, especially if they rely on a legitimate proxy.

### Modern Mitigations

If the client has SMB Signing set to "Required" or has disabled LLMNR/NetBIOS via GPO, Responder will still see the traffic but won't be able to relay or capture usable credentials effectively.
