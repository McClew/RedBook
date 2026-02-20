# Splunk Enumeration

Splunk is a log analytics tool used to gather, analyse and visualise data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics. Splunk deployments are often used to house sensitive data and could provide a wealth of information for an attacker if compromised.

{% hint style="info" %}
## Default Port

The Splunk web server runs by default on port 8000.
{% endhint %}

***

## Trial Version

The Splunk Enterprise trial converts to a free version after 60 days, which doesn’t require authentication. It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about.

This will automatically convert to the free version that **does not have any form of authentication**, introducing a security hole in the environment. Some organisations may opt for the free version due to budget constraints, not fully understanding the implications of having no user/role management.

***

## Discovery & Footprinting

We can discover Splunk with a quick Nmap service scan. Here we can see that Nmap identified the `Splunkd httpd` service on port 8000 and port 8089, the Splunk management port for communication with the Splunk REST API.

{% code title="Command" %}
```bash
sudo nmap -sV 10.129.201.50
```
{% endcode %}

{% code title="Output" %}
```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-22 08:43 EDT
Nmap scan report for 10.129.201.50
Host is up (0.11s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp open  ssl/http      Splunkd httpd
8080/tcp open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.22 seconds
```
{% endcode %}
