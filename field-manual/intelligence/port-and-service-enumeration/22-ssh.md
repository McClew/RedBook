---
icon: ethernet
---

# 22 - SSH

**SSH (Secure Shell or Secure Socket Shell)** is a network protocol that enables a secure connection to a computer over an unsecured network. It is essential for maintaining the confidentiality and integrity of data when accessing remote systems.

{% hint style="info" %}
Default port: 22
{% endhint %}

```bash
PORT   STATE SERVICE
22/tcp open  ssh     syn-ack
```

***

## Cheatsheet

| Action                                                                        | Description                                                                                                   |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `ssh-audit <target>`                                                          | Perform a security audit of the target SSH service, checking for vulnerabilities and misconfigurations.       |
| `ssh <user>@<target>`                                                         | Log in to the SSH server using the SSH client.                                                                |
| `ssh -i private.key <user>@<target>`                                          | Log in to the SSH server using a private key for authentication.                                              |
| `ssh <user>@<target> -o PreferredAuthentications=password`                    | Force password-based authentication for login.                                                                |
| `cat /etc/ssh/sshd_config \| grep -E 'PermitRootLogin\|PubkeyAuthentication'` | If you have a shell on the target, check if root login or public key authentication is enabled on the server. |

***

## Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

```bash
nc -vn <IP> 22
```

***

## Automated ssh-audit <a href="#automated-ssh-audit" id="automated-ssh-audit"></a>

ssh-audit is a tool for ssh server & client configuration auditing.

[https://github.com/jtesta/ssh-audit](https://github.com/jtesta/ssh-audit) is an updated fork from [https://github.com/arthepsy/ssh-audit/](https://github.com/arthepsy/ssh-audit/)

**Features:**

* SSH1 and SSH2 protocol server support;
* analyze SSH client configuration;
* grab banner, recognize device or software and operating system, detect compression;
* gather key-exchange, host-key, encryption and message authentication code algorithms;
* output algorithm information (available since, removed/disabled, unsafe/weak/legacy, etc);
* output algorithm recommendations (append or remove based on recognized software version);
* output security information (related issues, assigned CVE list, etc);
* analyze SSH version compatibility based on algorithm information;
* historical information from OpenSSH, Dropbear SSH and libssh;
* runs on Linux and Windows;
* no dependencies

```bash
usage: ssh-audit.py [-1246pbcnjvlt] <host>

   -1,  --ssh1             force ssh version 1 only
   -2,  --ssh2             force ssh version 2 only
   -4,  --ipv4             enable IPv4 (order of precedence)
   -6,  --ipv6             enable IPv6 (order of precedence)
   -p,  --port=<port>      port to connect
   -b,  --batch            batch output
   -c,  --client-audit     starts a server on port 2222 to audit client
                               software config (use -p to change port;
                               use -t to change timeout)
   -n,  --no-colors        disable colors
   -j,  --json             JSON output
   -v,  --verbose          verbose output
   -l,  --level=<level>    minimum output level (info|warn|fail)
   -t,  --timeout=<secs>   timeout (in seconds) for connection and reading
                               (default: 5)
$ python3 ssh-audit <IP>
```

***

## Public SSH key of server <a href="#public-ssh-key-of-server" id="public-ssh-key-of-server"></a>

```bash
ssh-keyscan -t rsa <IP> -p <PORT>
```

***

## Weak Cipher Algorithms <a href="#weak-cipher-algorithms" id="weak-cipher-algorithms"></a>

This is discovered by default by **nmap**. But you can also use **sslcan** or **sslyze**.

***

## Nmap Scripts <a href="#nmap-scripts" id="nmap-scripts"></a>

```bash
nmap -p22 <ip> -sC # Send default nmap scripts for SSH
nmap -p22 <ip> -sV # Retrieve version
nmap -p22 <ip> --script ssh2-enum-algos # Retrieve supported algorythms
nmap -p22 <ip> --script ssh-hostkey --script-args ssh_hostkey=full # Retrieve weak keys
nmap -p22 <ip> --script ssh-auth-methods --script-args="ssh.user=root" # Check authentication methods
```
