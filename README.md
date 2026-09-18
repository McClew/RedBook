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

# Field Kit

> ## Field Manual
>
> The Field Manual serves as a comprehensive and practical reference guide for conducting cybersecurity assessments and penetration tests. It has been designed and structured around the core phases of the **Penetration Testing Execution Standard (PTES)**, providing a systematic approach to security engagements.
>
> While incorporating some minor deviations for practical utility, the manual largely adheres to the PTES Technical Guidelines, which define a standardised methodology for penetration testing.
>
> For more detailed information on the foundational principles and technical aspects that underpin this manual's structure, please refer to the official PTES website: [http://www.pentest-standard.org/index.php/PTES\_Technical\_Guidelines](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines).
>
> This manual aims to distil complex methodologies and accumulated knowledge into an easily accessible format, enabling users to quickly locate and apply relevant information across all stages of a penetration testing engagement.

## CPTS Knowledge Base — Gap To-Do List

_Generated from a full review of the Red Book against the HTB Academy "Penetration Tester" path (28 modules). Covers fully-blank leaf pages and entirely missing topics only — blank section index pages are intentionally excluded._

### Penetration Testing Process

| Topic                | Type       | Path                                       | Priority | Status |
| -------------------- | ---------- | ------------------------------------------ | -------- | ------ |
| Attack Path Planning | Blank Page | field-manual/analysis/attack-path-planning | Medium   | To Do  |

### Footprinting

| Topic                         | Type       | Path                                                                            | Priority | Status |
| ----------------------------- | ---------- | ------------------------------------------------------------------------------- | -------- | ------ |
| 88 - Kerberos                 | Blank Page | field-manual/intelligence/port-and-service-enumeration/88-kerberos              | High     | To Do  |
| 135, 593 - MSRPC              | Blank Page | field-manual/intelligence/port-and-service-enumeration/135-593-msrpc            | High     | To Do  |
| 137, 138, 139 - NetBIOS       | Blank Page | field-manual/intelligence/port-and-service-enumeration/137-138-139-netbios      | High     | To Do  |
| 161, 162, 10161, 10162 - SNMP | Blank Page | field-manual/intelligence/port-and-service-enumeration/161-162-10161-10162-snmp | High     | To Do  |
| 3389 - RDP                    | Blank Page | field-manual/intelligence/port-and-service-enumeration/3389-rdp                 | High     | To Do  |
| 22 - SFTP                     | Blank Page | field-manual/intelligence/port-and-service-enumeration/22-sftp                  | Medium   | To Do  |
| 80, 443 - Web                 | Blank Page | field-manual/intelligence/port-and-service-enumeration/80-443-web               | Medium   | To Do  |
| 143, 993 - IMAP               | Blank Page | field-manual/intelligence/port-and-service-enumeration/143-993-imap             | Medium   | To Do  |
| 500 - IPsec/IKE VPN           | Blank Page | field-manual/intelligence/port-and-service-enumeration/500-ipsec-ike-vpn        | Medium   | To Do  |
| 873 - Rsync                   | Blank Page | field-manual/intelligence/port-and-service-enumeration/873-rsync                | Medium   | To Do  |
| 27017, 27018 - MongoDB        | Blank Page | field-manual/intelligence/port-and-service-enumeration/27017-27018-mongodb      | Medium   | To Do  |
| enum4linux                    | Blank Page | toolbox/tooling/information-gathering/linux-enumeration/enum4linux              | Medium   | To Do  |
| HUMINT                        | Blank Page | field-manual/intelligence/humint                                                | Low      | To Do  |
| 7 - Echo                      | Blank Page | field-manual/intelligence/port-and-service-enumeration/7-echo                   | Low      | To Do  |
| 43 - WHOIS                    | Blank Page | field-manual/intelligence/port-and-service-enumeration/43-whois                 | Low      | To Do  |
| 49 - TACACS+                  | Blank Page | field-manual/intelligence/port-and-service-enumeration/49-tacacs+               | Low      | To Do  |
| 79 - Finger                   | Blank Page | field-manual/intelligence/port-and-service-enumeration/79-finger                | Low      | To Do  |
| 111 - Portmapper              | Blank Page | field-manual/intelligence/port-and-service-enumeration/111-portmapper           | Low      | To Do  |
| 113 - Ident                   | Blank Page | field-manual/intelligence/port-and-service-enumeration/113-ident                | Low      | To Do  |
| 123 - NTP                     | Blank Page | field-manual/intelligence/port-and-service-enumeration/123-ntp                  | Low      | To Do  |
| 194, 6667, 6660-7000 - IRC    | Blank Page | field-manual/intelligence/port-and-service-enumeration/194-6667-6660-7000-irc   | Low      | To Do  |

### Network Enumeration with Nmap

| Topic   | Type       | Path                                                              | Priority | Status |
| ------- | ---------- | ----------------------------------------------------------------- | -------- | ------ |
| Masscan | Blank Page | toolbox/tooling/information-gathering/network-enumeration/masscan | Low      | To Do  |

### Vulnerability Assessment

| Topic                  | Type       | Path                                            | Priority | Status |
| ---------------------- | ---------- | ----------------------------------------------- | -------- | ------ |
| Vulnerability Scanning | Blank Page | field-manual/analysis/vulnerability-scanning    | High     | To Do  |
| Nuclei                 | Blank Page | toolbox/tooling/web-application-analysis/nuclei | Medium   | To Do  |

### Shells & Payloads

| Topic  | Type       | Path                                     | Priority | Status |
| ------ | ---------- | ---------------------------------------- | -------- | ------ |
| NetCat | Blank Page | toolbox/tooling/post-exploitation/netcat | Medium   | To Do  |

### Using the Metasploit Framework

| Topic    | Type       | Path                                                   | Priority | Status |
| -------- | ---------- | ------------------------------------------------------ | -------- | ------ |
| MSFVenom | Blank Page | toolbox/tooling/exploitation-tools/metasploit/msfvenom | High     | To Do  |

### Password Attacks

| Topic               | Type       | Path                                                          | Priority | Status |
| ------------------- | ---------- | ------------------------------------------------------------- | -------- | ------ |
| Password Extraction | Blank Page | field-manual/post-exploitation/password-extraction            | High     | To Do  |
| Custom Rules        | Blank Page | field-manual/post-exploitation/password-cracking/custom-rules | Medium   | To Do  |

### Login Brute Forcing

| Topic                                        | Type          | Path                                   | Priority | Status |
| -------------------------------------------- | ------------- | -------------------------------------- | -------- | ------ |
| Hydra                                        | Blank Page    | toolbox/tooling/password-attacks/hydra | High     | To Do  |
| Medusa (brute-force tool, complements Hydra) | Missing Topic | not yet created                        | Low      | To Do  |

### Pivoting, Tunneling, and Port Forwarding

| Topic       | Type       | Path                                      | Priority | Status |
| ----------- | ---------- | ----------------------------------------- | -------- | ------ |
| Proxychains | Blank Page | toolbox/tooling/network-tools/proxychains | High     | To Do  |
| Socat       | Blank Page | toolbox/tooling/network-tools/socat       | Medium   | To Do  |

### Active Directory Enumeration & Attacks

| Topic                                                                                   | Type          | Path                                                           | Priority | Status |
| --------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------------- | -------- | ------ |
| Lateral Movement                                                                        | Blank Page    | field-manual/post-exploitation/lateral-movement                | High     | To Do  |
| Mimikatz                                                                                | Blank Page    | toolbox/tooling/post-exploitation/mimikatz                     | High     | To Do  |
| PsExec                                                                                  | Blank Page    | toolbox/tooling/post-exploitation/psexec                       | High     | To Do  |
| Active Directory Certificate Services (ADCS) attacks (ESC1-ESC8, Certipy)               | Missing Topic | not yet created                                                | High     | To Do  |
| Kerberos Delegation Abuse (unconstrained / constrained / RBCD)                          | Missing Topic | not yet created                                                | High     | To Do  |
| DACL / ACL Abuse (GenericAll, GenericWrite, WriteDACL, ForceChangePassword, WriteOwner) | Missing Topic | not yet created                                                | High     | To Do  |
| NTLM Relay & Coercion Attacks (PetitPotam, PrintNightmare, ZeroLogon, noPac)            | Missing Topic | not yet created                                                | High     | To Do  |
| Golden Ticket / Silver Ticket Attacks                                                   | Missing Topic | not yet created                                                | High     | To Do  |
| Pass-the-Hash / Overpass-the-Hash                                                       | Missing Topic | not yet created                                                | High     | To Do  |
| DHCP DNS Injection                                                                      | Blank Page    | field-manual/exploitation/man-in-the-middle/dhcp-dns-injection | Medium   | To Do  |
| WPAD Poisoning                                                                          | Blank Page    | field-manual/exploitation/man-in-the-middle/wpad-poisoning     | Medium   | To Do  |
| Domain Trust Abuse (parent/child, cross-forest, SIDHistory/ExtraSIDs)                   | Missing Topic | not yet created                                                | Medium   | To Do  |
| Group Policy Object (GPO) Abuse                                                         | Missing Topic | not yet created                                                | Medium   | To Do  |

### Attacking Web Applications with Ffuf

| Topic     | Type       | Path                                            | Priority | Status |
| --------- | ---------- | ----------------------------------------------- | -------- | ------ |
| Gobuster  | Blank Page | toolbox/tooling/information-gathering/gobuster  | Medium   | To Do  |
| Dirbuster | Blank Page | toolbox/tooling/information-gathering/dirbuster | Low      | To Do  |

### SQL Injection Fundamentals

| Topic            | Type       | Path                                                   | Priority | Status |
| ---------------- | ---------- | ------------------------------------------------------ | -------- | ------ |
| NoSQLi Injection | Blank Page | field-manual/exploitation/web-attacks/nosqli-injection | Low      | To Do  |

### Cross-Site Scripting (XSS)

| Topic             | Type       | Path                                                                             | Priority | Status |
| ----------------- | ---------- | -------------------------------------------------------------------------------- | -------- | ------ |
| Reflected XSS     | Blank Page | field-manual/exploitation/web-attacks/cross-site-scripting-xss/reflected-xss     | High     | To Do  |
| Session Hijacking | Blank Page | field-manual/exploitation/web-attacks/cross-site-scripting-xss/session-hijacking | Medium   | To Do  |

### File Inclusion

| Topic                      | Type       | Path                                                         | Priority | Status |
| -------------------------- | ---------- | ------------------------------------------------------------ | -------- | ------ |
| File Inclusion (LFI / RFI) | Blank Page | field-manual/exploitation/web-attacks/file-inclusion-lfi-rfi | High     | To Do  |

### Web Attacks

| Topic                                    | Type       | Path                                                                         | Priority | Status |
| ---------------------------------------- | ---------- | ---------------------------------------------------------------------------- | -------- | ------ |
| Remote Code Execution                    | Blank Page | field-manual/exploitation/remote-code-execution                              | High     | To Do  |
| Insecure Direct Object References (IDOR) | Blank Page | field-manual/exploitation/web-attacks/insecure-direct-object-references-idor | High     | To Do  |
| Authentication Bypass                    | Blank Page | field-manual/exploitation/authentication-bypass                              | Medium   | To Do  |
| Authentication Vulnerabilities           | Blank Page | field-manual/exploitation/web-attacks/authentication-vulnerabilities         | Medium   | To Do  |
| Cross-Site Request Forgery (CSRF)        | Blank Page | field-manual/exploitation/web-attacks/cross-site-request-forgery-csrf        | Medium   | To Do  |
| Server-Side Template Injection (SSTI)    | Blank Page | field-manual/exploitation/web-attacks/server-side-template-injection-ssti    | Medium   | To Do  |
| Clickjacking                             | Blank Page | field-manual/exploitation/web-attacks/clickjacking                           | Low      | To Do  |
| Cross-Origin Resource Sharing (CORS)     | Blank Page | field-manual/exploitation/web-attacks/cross-origin-resource-sharing-cors     | Low      | To Do  |
| Web Cache Deception                      | Blank Page | field-manual/exploitation/web-attacks/web-cache-deception                    | Low      | To Do  |
| WebSocket Vulnerabilities                | Blank Page | field-manual/exploitation/web-attacks/websocket-vulnerabilities              | Low      | To Do  |

### Windows Privilege Escalation

| Topic               | Type       | Path                                                                      | Priority | Status |
| ------------------- | ---------- | ------------------------------------------------------------------------- | -------- | ------ |
| Legacy Exploitation | Blank Page | field-manual/post-exploitation/privilege-escalation-1/legacy-exploitation | Medium   | To Do  |

### Documentation & Reporting

| Topic            | Type       | Path                                                         | Priority | Status |
| ---------------- | ---------- | ------------------------------------------------------------ | -------- | ------ |
| Proof-of-Concept | Blank Page | field-manual/post-engagement/report-writing/proof-of-concept | High     | To Do  |
| QA Checklist     | Blank Page | field-manual/post-engagement/report-writing/qa-checklist     | High     | To Do  |
| Executive Report | Blank Page | field-manual/post-engagement/executive-report                | High     | To Do  |
| Technical Report | Blank Page | field-manual/post-engagement/technical-report                | High     | To Do  |

### Attacking Enterprise Networks

| Topic             | Type       | Path                                                   | Priority | Status |
| ----------------- | ---------- | ------------------------------------------------------ | -------- | ------ |
| Command & Control | Blank Page | field-manual/post-exploitation/command-and-control     | Medium   | To Do  |
| Firewall Bypass   | Blank Page | field-manual/post-exploitation/evasion/firewall-bypass | Medium   | To Do  |
| Persistence       | Blank Page | field-manual/post-exploitation/persistence             | Medium   | To Do  |

### Other / Not in current CPTS path

| Topic            | Type       | Path                                      | Priority | Status |
| ---------------- | ---------- | ----------------------------------------- | -------- | ------ |
| Buffer Overflow  | Blank Page | field-manual/exploitation/buffer-overflow | Low      | To Do  |
| Wireless Attacks | Blank Page | toolbox/tooling/wireless-attacks          | Low      | To Do  |
