# WPScan

WPScan is an automated WordPress scanner and enumeration tool. It determines if the various themes and plugins used by a blog are outdated or vulnerable.

{% hint style="info" %}
## Download & Install

**Recommended:**\
Install via Kali package manager: `sudo apt install wpscan`



Download and install from GitHub:\
[https://github.com/wpscanteam/wpscan](https://github.com/wpscanteam/wpscan)
{% endhint %}

***

## Basic Commands

### Enumeration

```bash
sudo wpscan --url http://blog.acme.com --enumerate
```

### Brute Force Attack

```bash
wpscan --url http://ir.inlanefreight.local -P passwords.txt -U ilfreightwp
```

***

## WPVulnDB API

Use an API token from [WPVulnDB](https://wpvulndb.com/), which is used by WPScan to scan for PoC and reports. The free plan allows up to 75 requests per day. Provide this token to WPScan using the `--api-token parameter`.
