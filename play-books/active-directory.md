---
icon: server
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

# Active Directory

## If / Then Guide

| If Enumeration Reveals... | ...Focus on these Attacks... | ...and perform these Actions |
| ------------------------- | ---------------------------- | ---------------------------- |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |
|                           |                              |                              |

***

## Uncredentialed Enumeration

### Host Indenticiation

{% stepper %}
{% step %}
#### Listening with Wireshark

Use Wireshark and listen for Layer 2 ( ARP, NDNS ) traffic to discover IP addresses and hostnames.
{% endstep %}

{% step %}
#### Analysing with Responder

Use Responder in 'Analyze' mode to discover IP addresses and hostnames
{% endstep %}

{% step %}
#### ICMP Sweep

Perform an Fping ICMP sweep to final all hosts on your subnet that respond to an ICMP echo request.
{% endstep %}

{% step %}
#### Nmap Network Scan

Perform an NMAP identification scan to validate findings (it may find something previously missed).
{% endstep %}

{% step %}
#### Nmap Service Scan

Use all discovered hosts to perform an NMAP scan to determine services running on each host.

1. Look for quick wins for initial foothold, like outdated software, service or OS.
{% endstep %}
{% endstepper %}

### User Identification

{% stepper %}
{% step %}
#### SMB NULL Session

Attempt to abuse an SMB NULL session against the domain controller.

1. rcpclient or enum4linux to grab users.
{% endstep %}

{% step %}
#### LDAP Anonymous Authentication

Attempt to abuse an anonymous LDAP search against the domain controller.

1. ldapsearch or windapsearch to grab users.
{% endstep %}

{% step %}
#### ASREP-Roasting

Check for users that do not require Kerberos pre-auth (ASREP-Roasting) to get password hash.
{% endstep %}

{% step %}
#### Kerbrute

Use Kerbrute and wordlists to brute force usernames against DC.
{% endstep %}

{% step %}
#### Impacket

Use Impackets lookupsid.py to discover users (works better with creds)
{% endstep %}

{% step %}
#### Enum

Look for systems that can be exploited to gain SYSTEM level access.
{% endstep %}
{% endstepper %}

### User Foothold

{% stepper %}
{% step %}
#### Responder Analysis

Use Responder/Inveigh on network interface to listen for NTLM users and hashes.

1. Attempt to crack hashes.
{% endstep %}

{% step %}
#### Password Spray

Attempt password spray on users identified during user identification.

1. attempt to gather password policy for organisation.
2. password spray using common passwords.
{% endstep %}
{% endstepper %}

***

## Credentialed Enumeration &  Exploitation

### Host Identification

{% stepper %}
{% step %}
#### Bloodhound

Run [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention") with discovered credentials.
{% endstep %}

{% step %}
#### LDAP Domain Dump

Use `ldapdomaindump` to identify all domain joined computers.
{% endstep %}

{% step %}
#### Share Enumeration

Enumerate accessible shares on servers with [crackmapexec.md](../toolbox/tooling/post-exploitation/crackmapexec.md "mention"), [smbmap.md](../toolbox/tooling/information-gathering/service-enumeration/smbmap.md "mention"), [powerview.md](../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention") or [snaffler.md](../toolbox/tooling/post-exploitation/snaffler.md "mention").
{% endstep %}
{% endstepper %}

### User Identificaiton

{% stepper %}
{% step %}
#### Bloodhound

Run Bloodhound with discovered credentials

1. Bloodhound.py
2. Bloodhound from windows
{% endstep %}

{% step %}
#### Password Policy Enumeration

Gather the domain password policy using the discovered credentails

1. Gather a list of Domain Admins or Privileged users using tools:
   1. Windapsearch
   2. powerview
   3. bloodhound
   4. ad powershell module
{% endstep %}
{% endstepper %}

### Foothold Enumeration

{% stepper %}
{% step %}
Enumerate security controls.
{% endstep %}

{% step %}
Look for other logged-in users using CME.
{% endstep %}

{% step %}
Look for Kerberoastable accounts.
{% endstep %}

{% step %}
Look at owned users for abusable ACL entries.
{% endstep %}

{% step %}
Check bloodhound for CanRDP, CanPSRemote, or SQLAdmin abilities for lateral movement.
{% endstep %}
{% endstepper %}

### Pivoting

{% stepper %}
{% step %}
Run chisel for windows on the windows pivot host

1. connect with the linux chisel client on the attack host.
2. modify the proxychain.conf file to match the proxy that is established.
3. run command with proxychains
{% endstep %}

{% step %}
Check bloodhound for CanRDP, CanPSRemote or SQLAdmin abilities forlateral movement.
{% endstep %}
{% endstepper %}

***

## Exploitation

{% stepper %}
{% step %}
look for kerberoastable accounts suing bloodhound, powerview, getuserspns.py
{% endstep %}

{% step %}
grab all TGS tickets with GetUserSPNs.py, save to file and attempt to crack.
{% endstep %}

{% step %}
Abuse any over permissive acl entries to gain control of more users and move laterally throughout the network.

1. ForceChangePassword
2. AddMember
3. GeneralAll/GenericWrite
4. Ds-Replpciation-GetChanges-All
5. ACL Abuse
{% endstep %}

{% step %}
check for common vulnerabilities:

1. NoPac
2. PrintNightmate
3. PetitPotam
{% endstep %}

{% step %}
Check for common misconfigurations to escalate privileges:

1. exchange group permissions
2. ms-rprn printer bug
3. MS14-068
4. sniff for LDAP credentials
5. enumerate DNS records for interesting servers
6. look for user passwords and other notes in AD user descriptions
7. check for PASSWD\_NOTRREQD field on users and test for weak/no passwoords.
8. Look for credentials and other interesting files on SMB shares.
9. check for DONT\_REQ\_PREAUTH field and ASREPRoasting any discovered users.
10. Check for GPOs that we have write access over to gain administrator rights or more latterally.
11. Resource based constrained delegation, constrained delegation, unconstrained delegation
12. active directory certificate services attacks
{% endstep %}

{% step %}
check for group policy regerences GPP passwords.
{% endstep %}
{% endstepper %}

### Additional Auditing

{% stepper %}
{% step %}
create a snapshot of the AD database with AD explorer for offline analysis
{% endstep %}

{% step %}
use PingCastle to discover additional AD misconfigurations and vulnerabilities
{% endstep %}

{% step %}
run group3r to uncover vulnerabilities in AD Group Policy.
{% endstep %}

{% step %}
Run ADRecon.ps1 to disocver additional AD misconfigurations and vulnerabilties that may have been missed.
{% endstep %}
{% endstepper %}

### Attacking AD Trusts (Parent Domain)

{% stepper %}
{% step %}
Discover any current domain trusts with other domains using Get-ADTrust, Get-DomainTrust (PowerView) or Bloodhound.
{% endstep %}

{% step %}
From a machine with Domain Admin privileges, attempt an ExtraSIDs attack to create an Enterprise admin user in the parent domain.
{% endstep %}

{% step %}
Domain trusts overview
{% endstep %}

{% step %}
child > parent attacks Windows
{% endstep %}

{% step %}
child > parent attacks Linux
{% endstep %}
{% endstepper %}

### Attacking AD Trusts (Cross Forest)

{% stepper %}
{% step %}
Discover any current domain trusts with other domains using Get-ADTrust, Get-DomainTrust (powerview) or bloodhound
{% endstep %}

{% step %}
Attempt cross-forsest kerberoasting
{% endstep %}

{% step %}
if admin accounts sharee names across domains, and on is compromsied, try credential reuse.
{% endstep %}

{% step %}
Check for SIDHistory abuse.
{% endstep %}

{% step %}
cross-forest trust abuse - windows
{% endstep %}

{% step %}
crossforst trust abuse - linux.
{% endstep %}
{% endstepper %}

***

## Additional Methods

{% stepper %}
{% step %}
Access a host via RDP or WinRM as a local user or a local admin.
{% endstep %}

{% step %}
Authenticate to a remote host as an admin using tools such as PsExec.
{% endstep %}

{% step %}
Gain access to a sensitive file share.
{% endstep %}

{% step %}
Gain MSSQL access to a host as a DBA user, which can then be leveraged to escalate permissions.
{% endstep %}
{% endstepper %}
