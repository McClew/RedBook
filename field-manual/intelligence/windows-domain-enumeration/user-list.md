---
icon: user
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

# Domain User & Group Enumeration

## Cheatsheet

{% tabs %}
{% tab title="Without Access" %}
| Command                                                                                                                                                                                                                                                                                                      | Description                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p><code>nxc smb &#x3C;dc> --users</code><br><code>nxc smb &#x3C;dc> -u '' -p '' --users</code></p>                                                                                                                                                                                                          | <p>SMB: Attempts to enumerate users via an SMB NULL session.<br><br><em>Only works if the target allows SMB NULL sessions.</em></p>                                                                                                                                                  |
| <p><code>ldapsearch -H ldap://&#x3C;dc> -x -b "&#x3C;domain-dn>" -s sub "(&#x26;(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "</code><br><br><br>The Distinguished Name (DN) for the domain follows a structure like this:<br>Domain: <code>BRM.COM</code> → DN: <code>DC=BRM,DC=COM</code></p> | <p>LDAP: Attempts to enumerate users via an anonymous LDAP bind.<br><br><em>Only works if anonymous binding is enabled (rare).</em></p>                                                                                                                                              |
| <p><code>nxc smb &#x3C;target> -u '' -p '' --rid-brute --rid-brute &#x3C;max_rid></code><br><code>nxc smb &#x3C;target> -u 'guest' -p '' --rid-brute --rid-brute &#x3C;max_rid></code></p>                                                                                                                   | <p><a data-mention href="../../../toolbox/tooling/exploitation-tools/netexec.md">netexec.md</a> (brute-force): Uses RID brute forcing to enumerate domain objects.<br><br>Defaults to RIDs up to 4000; using 8000+ is recommended for better coverage.</p>                           |
| <p><code>kerbrute userenum -d &#x3C;domain> --dc &#x3C;dc> &#x3C;wordlist> -o &#x3C;output-file></code><br><br>Copy the output to a file, then extract users with:<br><code>sed -n 's/.*VALID USERNAME:[[:space:]]*\([^@]*\)@.*/\1/p' output.txt > users.lst</code></p>                                      | <p>Kerbrute (brute-force): Uses Kerbrute and a worslist to enumerate valid usernames via Kerberos pre-authentication.<br><br>Does not trigger account lockouts.<br><br>Try to determine the username format and find a suitable wordlist.</p>                                        |
| <p><code>sudo responder -r</code> </p><p><code>-I &#x3C;network-interface></code></p>                                                                                                                                                                                                                        | <p><a data-mention href="../../../toolbox/tooling/sniffing-and-spoofing/responder.md">responder.md</a>: Launches Responder with default settings.<br><br>Intercepts LLMNR/NBT-NS requests to capture usernames and password hashes.<br><br>Usernames must be extracted manually.</p> |
{% endtab %}

{% tab title="With Access (windows)" %}
<table><thead><tr><th width="326">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>Get-NetDomain</code></td><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a>: Retrieves basic information about the current Active Directory domain.</td></tr><tr><td><p>All information:<br><code>Get-NetUser</code><br>Only crucial information:</p><p><code>Get-NetUser | select</code> </p><p><code>cn,pwdlastset,lastlogon</code></p></td><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a>: Lists all domain users, including details like password last set and last logon time.</td></tr><tr><td>All information:<br><code>Get-NetGroup</code><br>Only crucial information:<br><code>Get-NetGroup | select cn</code></td><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a>: Enumerates all domain groups.</td></tr><tr><td><code>Get-NetUser -SPN | select samaccountname,serviceprincipalname</code></td><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a>: Finds accounts with Service Principal Names (SPNs), useful for Kerberoasting.</td></tr><tr><td><p><code>Get-NetSession -Verbose</code> </p><p><code>-ComputerName &#x3C;cn></code></p></td><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a><strong>:</strong> Lists active user sessions on a remote computer (requires local admin rights).<br><br>This could be a valuable opportunity to steal domain admin credentials from memory or impersonate them.</td></tr><tr><td>All users:<br><code>net user /domain</code><br>Specific user:<br><code>net user &#x3C;user> /domain</code></td><td>CMD: Lists all domain users or detailed info for a specific user.</td></tr><tr><td>All groups:<br><code>net group /domain</code><br>Specific group:<br><code>net group &#x3C;group> /domain</code></td><td>CMD: Lists all domain groups or members of a specified group.</td></tr></tbody></table>
{% endtab %}

{% tab title="With Access (Linux)" %}
<table><thead><tr><th width="343">Command</th><th>Description</th></tr></thead><tbody><tr><td><p><code>nxc smb &#x3C;dc-ip> -u &#x3C;user></code> </p><p><code>-p &#x3C;password> --users</code><br><br>Copy the output to a file, then extract users with:<br><code>awk '$5 ~ /^[a-zA-Z0-9_]+$/ &#x26;&#x26; NF >= 5 { print $5 }' output.txt > users.lst</code></p></td><td><strong>(SMB)</strong> Retrieves a list of all users in the domain.<br><br>Also shows the count of bad password attempts for each user.<br><br></td></tr><tr><td><p><code>nxc smb &#x3C;dc-ip> -u &#x3C;user></code> </p><p><code>-p &#x3C;password> --groups</code></p></td><td><strong>(SMB)</strong> Retrieves a list of all groups in the domain.<br><br>Includes the member count for each group.<br><br>Pay special attention to key groups such as:<br>- Administrators<br>- Domain Admins<br>- Executives</td></tr><tr><td><code>nxc smb &#x3C;host> -u &#x3C;user> -p &#x3C;password> --loggedon-users</code></td><td><strong>(SMB)</strong> Lists users currently logged on to the specified host (requires local admin rights).<br><br>This could be a valuable opportunity to steal domain admin credentials from memory or impersonate them.</td></tr></tbody></table>
{% endtab %}
{% endtabs %}

***

## SMB NULL Session

### Tools

<table><thead><tr><th width="196">Name</th><th>Description</th></tr></thead><tbody><tr><td><a data-mention href="../../../toolbox/tooling/post-exploitation/crackmapexec.md">crackmapexec.md</a></td><td>An open-source tool that automates the assessment of Active Directory networks by leveraging various protocols to perform reconnaissance, credential gathering, lateral movement, and exploitation.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/information-gathering/service-enumeration/rpcclient.md">rpcclient.md</a></td><td>A command-line tool, part of the Samba suite, used to connect to and interact with Windows SMB/MS-RPC services, primarily for administration and network enumeration of users, groups, shares, and other domain information.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/information-gathering/linux-enumeration/enum4linux.md">enum4linux.md</a></td><td>A Perl script that wraps around various Samba tools (like <code>smbclient</code> and <code>rpcclient</code>) to automate information gathering from Windows and Samba systems, including users, groups, shares, and password policies.</td></tr></tbody></table>

### enum4linux

We can use the `enumdomusers` command after connecting anonymously using `rpcclient`.

```bash
clue@machine[~]$ enum4linux -U $IP  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch
ccruz
njohnson
mholliday
<SNIP>
```

### rpcclient

Use `CrackMapExec` with the `--users` flag. This is a useful tool that will also show the `badpwdcount` (invalid login attempts), so we can remove any accounts from our list that are close to the lockout threshold.

It also shows the `baddpwdtime`, which is the date and time of the last bad password attempt, so we can see how close an account is to having its `badpwdcount` reset.

```bash
clue@machine[~]$ rpcclient -U "" -N $IP

rpcclient $> enumdomusers 
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]

<SNIP>
```

{% hint style="info" %}
In an environment with multiple Domain Controllers, this value is maintained separately on each one. To get an accurate total of the account's bad password attempts, we would have to either query each Domain Controller and use the sum of the values or query the Domain Controller with the PDC Emulator FSMO role.
{% endhint %}

### CrackMapExec

```bash
clue@machine[~]$ crackmapexec smb $IP --users

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 0 baddpwdtime: 2022-02-17 22:59:22.684613

<SNIP>
```

***

## LDAP Anonymous Bind

### Tools

<table><thead><tr><th width="213">Name</th><th>Description</th></tr></thead><tbody><tr><td>ldapsearch</td><td>A command-line utility for querying LDAP directories, including Active Directory, to retrieve specific entries and attributes based on defined filters and scope.</td></tr><tr><td>windapsearch.py</td><td>A Python script for enumerating users, groups, computers, and other objects from Active Directory domains by querying the LDAP protocol.</td></tr></tbody></table>

### ldapsearch

```bash
clue@machine[~]$ ldapsearch -h $IP -x -b "DC=ACME,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

guest
ACME-EA-DC01$
ACME-EA-MS01$
ACME-EA-WEB01$
student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch

<SNIP>
```

{% hint style="info" %}
When using `ldapsearch` you need to specify a valid LDAP search filter.
{% endhint %}

### windapsearch

```bash
clue@machine[~]$ ./windapsearch.py --dc-ip $IP -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

cn: Fae Anthony
userPrincipalName: fanthony@inlanefreight.local

cn: Walter Dillard
userPrincipalName: wdillard@inlanefreight.local

<SNIP>
```

{% hint style="info" %}
Specify anonymous access by providing a blank username with the `-u` flag and the `-U` flag to tell the tool to retrieve just users.
{% endhint %}

***

## Kerberose Pre-Authentication

### Kerbrute

If you have no access at all to our internal network, we can use `Kerbrute` to enumerate valid AD accounts and for password spraying.

The tool sends TGT requests to the domain controller without Kerberos Pre-Authentication to perform username enumeration. If the KDC responds with the error `PRINCIPAL UNKNOWN`, the username is invalid. Whenever the KDC prompts for Kerberos Pre-Authentication, this signals that the username exists, and the tool will mark it as valid.

{% hint style="info" %}
This method of username enumeration does not generate Windows event ID [4625: An account failed to log on](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625), or a logon failure which is often monitored for, and will not lock out accounts.
{% endhint %}

```bash
clue@machine[~]$  kerbrute userenum -d acme.local --dc $IP /opt/jsmith.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 tjohnson@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jwilson@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 bdavis@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 njohnson@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 asanchez@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 dlewis@acme.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 ccruz@acme.local

<SNIP>
```

{% hint style="warning" %}
Using Kerbrute for username enumeration will generate event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768). This will only be triggered if [Kerberos event logging](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging) is enabled via Group Policy.
{% endhint %}

{% hint style="success" %}
&#x20;Defenders can tune their SIEM tools to look for an influx of this event ID, which may indicate an attack. If we are successful with this method during a penetration test, this can be an excellent recommendation to add to our report.
{% endhint %}

***

## Credentialed Enumeration

### Tools

<table><thead><tr><th width="196">Name</th><th>Description</th></tr></thead><tbody><tr><td><a data-mention href="../../../toolbox/tooling/post-exploitation/crackmapexec.md">crackmapexec.md</a></td><td>An open-source tool that automates the assessment of Active Directory networks by leveraging various protocols to perform reconnaissance, credential gathering, lateral movement, and exploitation.</td></tr></tbody></table>

### CrackMapExec

```bash
clue@machine[~]$ sudo crackmapexec smb $IP -u acmeadmin -p Acme_admin_AD! --users

[sudo] password for acmeadmin: 
SMB         172.16.5.5      445    ACME-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACME-EA-DC01) (domain:ACME.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACME-EA-DC01  [+] INLANEFREIGHT.LOCAL\acmeadmin:Acme_admin_AD! 
SMB         172.16.5.5      445    ACME-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\administrator                  badpwdcount: 1 baddpwdtime: 2022-02-23 21:43:35.059620
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\acmeadmin                      badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\avazquez                       badpwdcount: 20 baddpwdtime: 2022-02-17 22:59:22.684613
SMB         172.16.5.5      445    ACME-EA-DC01  ACME.LOCAL\pfalcon                        badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
<SNIP>
```
