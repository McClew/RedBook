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
  actions:
    visible: true
  anchors:
    visible: true
---

# Proxychains

Proxychains forces the network traffic of any dynamically linked TCP application through a chain of one or more proxies — SOCKS4, SOCKS5 or HTTP — without needing native proxy support in the application itself. It's the tool we reach for once we have a SOCKS listener up (via Chisel or Ligolo-ng, or a plain SSH dynamic port forward) and need to run ordinary tools like `nmap`, `smbclient` or `xfreerdp` through it.

{% hint style="info" %}
#### Download & Install

[https://github.com/haad/proxychains-ng](https://github.com/haad/proxychains-ng)

Usually already installed on Kali, or available via `apt install proxychains4`.
{% endhint %}

***

## Configuration

The configuration file lives at `/etc/proxychains.conf` (or `/etc/proxychains4.conf` on some distributions). It's often worth keeping a separate copy per engagement and pointing at it with `-f`, rather than editing the system-wide file.

### Chain Types

Only one of these should be uncommented at a time, near the top of the file.

<table><thead><tr><th width="180">Chain type</th><th>Behaviour</th></tr></thead><tbody><tr><td><code>dynamic_chain</code></td><td>Uses every live proxy in the list, in order, skipping any that are dead. The usual choice.</td></tr><tr><td><code>strict_chain</code></td><td>Uses every proxy in the list, in order, and fails outright if any one of them is unreachable.</td></tr><tr><td><code>random_chain</code></td><td>Picks a random proxy from the list for each new connection.</td></tr></tbody></table>

{% hint style="info" %}
#### Note

`dynamic_chain` is the most forgiving option and works fine even when we only have a single proxy defined, which is the most common case when pivoting through one compromised host.
{% endhint %}

### Adding a Proxy

Proxies go under the `[ProxyList]` section at the bottom of the file, one per line, as `<type> <host> <port>`.

{% code title="/etc/proxychains.conf" %}
```
[ProxyList]
socks5 127.0.0.1 1080
```
{% endcode %}

This is the entry we'd add after standing up a SOCKS listener on port `1080` — whether that's Chisel, Ligolo-ng, or an SSH `-D` dynamic forward.

***

## Basic Usage

Once a proxy is defined, prefix whichever command we want to run with `proxychains` (or `proxychains4`, depending on the distribution).

{% stepper %}
{% step %}
### Confirm the Proxy is Listening

```bash
clue@attacker[~]$ ss -ntlp | grep 1080

LISTEN 0 128 127.0.0.1:1080 0.0.0.0:*
```
{% endstep %}

{% step %}
### Run a Command Through the Chain

```bash
clue@attacker[~]$ proxychains nmap -sT -Pn -p445,3389 172.16.5.19

ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:1080-<><>-172.16.5.19:445-<><>-OK
|S-chain|-<>-127.0.0.1:1080-<><>-172.16.5.19:3389-<><>-OK
```

{% hint style="warning" %}
Proxychains can't proxy raw sockets, so scan types relying on them — `-sS`, `-sU`, OS detection, and so on — won't work. Use `-sT` (a full TCP connect scan) for compatibility.
{% endhint %}
{% endstep %}

{% step %}
### Pivot with a Full Client

```bash
clue@attacker[~]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

Any dynamically linked TCP client can be chained this way, including `smbclient`, `mysql`, `psql`, and even a browser such as `firefox` for reaching an internal web application.
{% endstep %}
{% endstepper %}

***

## Common Pitfalls

{% hint style="danger" %}
#### DNS Leaks

DNS lookups aren't proxied by default and can leak straight from our attack host, or simply fail to resolve internal-only hostnames. Uncomment `proxy_dns` in the configuration file to force resolution through the chain as well.
{% endhint %}

{% hint style="warning" %}
#### Timeouts

Chained connections are inherently slower than direct ones. If we're seeing frequent timeouts with `nmap`, drop the timing template down a notch (`-T2`) rather than fighting Proxychains' own `tcp_read_time_out` / `tcp_connect_time_out` values.
{% endhint %}

***

## Resources

[https://github.com/haad/proxychains-ng](https://github.com/haad/proxychains-ng)
