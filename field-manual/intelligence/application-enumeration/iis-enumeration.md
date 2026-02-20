# IIS Enumeration

IIS stands for **Internet Information Services**, a _Microsoft web server_ used to host, deploy, and manage web applications and content on Windows operating systems.

## Enumeration

### Port Scanning

{% code title="Nmap Command" %}
```bash
nmap -p- -sV -sC --open 10.129.224.91
```
{% endcode %}

{% code title="Output" %}
```
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-14 19:44 GMT
Nmap scan report for 10.129.224.91
Host is up (0.011s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.38 seconds
```
{% endcode %}

IIS 7.5 is running on port 80. Executing a [#iis-tilde](../../exploitation/attacking-applications/iss-exploitation.md#iis-tilde "mention") enumeration attack on this version could be a viable option.
