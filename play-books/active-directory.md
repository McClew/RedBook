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

| If Enumeration Reveals...                                                | ...Focus on these Attacks...                                                                                                                                                                                                                                                                      | ...and perform these Actions                                                                                               |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| SMB NULL session / LDAP anonymous bind allowed                           | <ul><li><a data-mention href="../field-manual/exploitation/attacking-services/ldap.md">ldap.md</a></li><li><a data-mention href="../field-manual/exploitation/attacking-services/smb.md">smb.md</a></li></ul>                                                                                     | Pull the user list, then feed it into AS-REP roasting and password spraying. `rpcclient -U "" -N <dc-ip>` → `enumdomusers` |
| Multicast/broadcast name resolution active (LLMNR/NBT-NS/mDNS)           | <ul><li><a data-mention href="../field-manual/exploitation/man-in-the-middle/llmnr-nbt-ns-poisoning.md">llmnr-nbt-ns-poisoning.md</a></li><li><a data-mention href="../field-manual/exploitation/man-in-the-middle/ipv6-dns-takeover.md">ipv6-dns-takeover.md</a></li></ul>                       | Poison with responder to capture NetNTLMv2, then crack or relay                                                            |
| SMB signing not required on hosts                                        | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/ntlm-relay-and-coercion-attacks.md">ntlm-relay-and-coercion-attacks.md</a></li></ul>                                                                                                                                    | Relay captured/coerced auth with `ntlmrelayx.py` to a non-signing host                                                     |
| Users without Kerberos pre-auth (`DONT_REQ_PREAUTH`)                     | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/as-rep-roasting.md">as-rep-roasting.md</a></li></ul>                                                                                                                                                                    | `impacket-GetNPUsers` to pull the AS-REP hash, crack with hashcat (`-m 18200`)                                             |
| Service accounts with an SPN (valid creds held)                          | <ul><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation-1/kerberoasting/">kerberoasting</a></li></ul>                                                                                                                                                                | `impacket-GetUserSPNs ... -request`, crack with hashcat (`-m 13100`)                                                       |
| Weak/guessable domain password policy                                    | <ul><li><a data-mention href="../field-manual/exploitation/brute-force/">brute-force</a> > <a data-mention href="../field-manual/exploitation/brute-force/smb.md">smb.md</a></li></ul>                                                                                                            | Password spray the user list: `nxc smb <dc-ip> -u users.txt -p 'Season2025!' --continue-on-success`                        |
| Abusable ACLs in BloodHound (GenericAll, WriteDACL, ForceChangePassword) | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/dacl-acl-abuse.md">dacl-acl-abuse.md</a></li></ul>                                                                                                                                                                      | Reset a password / add to group / grant DCSync via the ACE, then pivot to that principal                                   |
| Owned principal has DS-Replication rights                                | <ul><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation-1/dcsync.md">dcsync.md</a></li></ul>                                                                                                                                                                         | `impacket-secretsdump` to DCSync the domain hashes (target `krbtgt`, Administrator)                                        |
| Delegation configured (unconstrained / constrained / RBCD)               | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/kerberos-delegation-abuse.md">kerberos-delegation-abuse.md</a></li></ul>                                                                                                                                                | Abuse the delegation to impersonate a privileged user to a service                                                         |
| AD CS / certificate templates present                                    | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/adcs-attack.md">adcs-attack.md</a></li></ul>                                                                                                                                                                            | `certipy find -vulnerable`, then exploit the matching ESCx path                                                            |
| Writable GPO / GPP cpassword in SYSVOL                                   | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/group-policy-object-gpo-abuse.md">group-policy-object-gpo-abuse.md</a></li></ul>                                                                                                                                        | Decrypt GPP cpassword, or abuse GPO write to run a privileged action                                                       |
| NThash or ticket obtained (no plaintext)                                 | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/pass-the-hash-overpass-the-hash.md">pass-the-hash-overpass-the-hash.md</a></li></ul>                                                                                                                                    | `nxc smb <target> -u user -H <nthash>`, or pass-the-ticket                                                                 |
| Domain/forest trust present                                              | <ul><li><a data-mention href="../field-manual/exploitation/initial-access/domain-trust-abuse.md">domain-trust-abuse.md</a></li><li><a data-mention href="../field-manual/exploitation/initial-access/golden-ticket-silver-ticket-attacks.md">golden-ticket-silver-ticket-attacks.md</a></li></ul> | Enumerate trust direction, then ExtraSID (child→parent) or cross-forest Kerberoast / SID-history                           |

***

## Uncredentialed Enumeration

### Host Identification

{% stepper %}
{% step %}
#### Passive Listening (Wireshark / net)

Passively listen for Layer 2 traffic (ARP, LLMNR/NBT-NS/mDNS) to discover IP addresses and hostnames without sending a single packet — useful when you must stay quiet.

```bash
sudo wireshark   # display filter: arp || llmnr || nbns || mdns
```

<mark style="color:$primary;">**What to look for:**</mark> broadcast name-resolution requests that reveal hostnames, the AD domain name (often in NBNS/Browser traffic), and which hosts are chatty.

```
NBNS  10.10.10.52 -> broadcast   Name query NB WORKSTATION01<00>
MDNS  10.10.10.60 -> 224.0.0.251 Standard query 0x0 PTR _smb._tcp.local
```

<mark style="color:$primary;">**Next:**</mark> record every hostname/IP into your target list; chatty hosts issuing failed lookups are prime poisoning targets → _Responder Analysis_.

* [ ] Complete
{% endstep %}

{% step %}
#### Analysing with Responder

Run [responder.md](../toolbox/tooling/sniffing-and-spoofing/responder.md "mention") in '**analyze mode**' - it _watches_ LLMNR/NBT-NS/mDNS without poisoning, so we can see what is poisonable before touching anything.

See [llmnr-nbt-ns-poisoning.md](../field-manual/exploitation/man-in-the-middle/llmnr-nbt-ns-poisoning.md "mention").

```bash
sudo responder -I eth0 -A
```

<mark style="color:$primary;">**What to look for:**</mark> `[Analyze mode: ...]` lines showing hosts requesting names that don't resolve - each is a host you could poison to capture its NetNTLMv2 hash. Note the requesting IP and the name requested.

```
[Analyze mode: LLMNR] Request by ['10.10.10.52'] for 'fileshare', ignoring
[Analyze mode: NBT-NS] Request by ['10.10.10.52'] for 'INTRANET', ignoring
```

<mark style="color:$primary;">**What it means:**</mark> a workstation is looking for a name with no DNS record (typo'd share, decommissioned host). In active mode you would answer as that host and the client would authenticate to you.

<mark style="color:$primary;">**Next:**</mark> if you see requests, drop 'analyze mode' and poison for real > _User Foothold › Responder / Inveigh Capture_. Separately, build a relay target list of hosts **without** SMB signing: `nxc smb 10.10.10.0/24 --gen-relay-list relay.txt`.

* [ ] Complete
{% endstep %}

{% step %}
#### ICMP Sweep

Perform an `fping` sweep to find hosts that respond to ICMP echo on the subnet.

```bash
fping -asgq 10.10.10.0/24
```

<mark style="color:$primary;">**What to look for:**</mark> the list of live IPs printed by `-a` (alive).

```
10.10.10.5
10.10.10.25
10.10.10.52
```

<mark style="color:$primary;">**Next:**</mark> feed the alive list into the nmap scans below. Note that ICMP is often filtered - a sparse result does **not** mean few hosts; confirm with nmap `-Pn`.

* [ ] Complete
{% endstep %}

{% step %}
#### Nmap Network Scan

Validate the live-host list with nmap - it may surface hosts the ICMP sweep missed.

```bash
nmap -sn 10.10.10.0/24 -oA hosts
```

<mark style="color:$primary;">**What to look for:**</mark> `Host is up` entries and any resolved hostnames.

```bash
Nmap scan report for DC01.domain.local (10.10.10.5)
Host is up (0.012s latency).
```

<mark style="color:$primary;">**Next:**</mark> take the confirmed host list into the service scan.

* [ ] Complete
{% endstep %}

{% step %}
#### Nmap Service Scan

Scan each host with nmap to fingerprint services. See [88-kerberos.md](../field-manual/intelligence/port-and-service-enumeration/88-kerberos.md "mention").

```bash
nmap -p- -sV -sC -oA ad_full <target>
```

<mark style="color:$primary;">**What to look for:**</mark> the DC "tell" - [88-kerberos.md](../field-manual/intelligence/port-and-service-enumeration/88-kerberos.md "mention")**,** [389-636-3268-3269-ldap.md](../field-manual/intelligence/port-and-service-enumeration/389-636-3268-3269-ldap.md "mention")**,** [139-445-smb.md](../field-manual/intelligence/port-and-service-enumeration/139-445-smb.md "mention")**,** [53-dns.md](../field-manual/intelligence/port-and-service-enumeration/53-dns.md "mention") open on one host confirms a domain controller. The LDAP/SMB scripts usually leak the domain FQDN and hostname.

```
88/tcp   open  kerberos-sec
389/tcp  open  ldap      Microsoft Windows AD LDAP (Domain: domain.local)
445/tcp  open  microsoft-ds
```

<mark style="color:$primary;">**What it means:**</mark> you now have the DC IP and the domain name - the two values that seed every command below.

<mark style="color:$primary;">**Next:**</mark> set your `<dc-ip>` and `domain.local` variables; check versions for quick-win RCE (see _Vulnerability Check_ below).
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
