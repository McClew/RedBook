---
icon: key-skeleton
---

# Domain Password Policy

## Cheatsheet

{% tabs %}
{% tab title="Authenticated" %}
<table data-full-width="true"><thead><tr><th width="333">Command</th><th>Description</th></tr></thead><tbody><tr><td><p><code>nxc smb &#x3C;dc> -u &#x3C;user> -p &#x3C;password></code></p><p> <code>--pass-pol</code></p></td><td><strong>(Linux)</strong> Retrieves password policy from a domain controller using SMB via NetExec.</td></tr><tr><td><code>Get-DomainPolicy</code></td><td><strong>(PowerView)</strong> Retrieves domain-wide password and Kerberos policy from Active Directory.</td></tr><tr><td><code>net accounts</code></td><td><strong>(CMD)</strong> Displays local password and account lockout policies on a Windows host.</td></tr></tbody></table>
{% endtab %}

{% tab title="SMB NULL Sessions" %}
<table><thead><tr><th width="259">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>nxc smb &#x3C;dc> -pass-pol</code></td><td><strong>(Linux)</strong> Retrieves domain password policy via SMB NULL session, if allowed, using NetExec.</td></tr><tr><td><code>rpcclient -U "" -N &#x3C;dc></code><br><code>querydominfo</code><br><code>getdompwinfo</code></td><td><strong>(Linux)</strong> Uses SMB NULL session, if enabled, to query domain and password policy info via <code>rpcclient</code>.</td></tr></tbody></table>
{% endtab %}

{% tab title="LDAP Anonymous Binds" %}
| Command                                                                                                                                                                                                                                                                     | Description                                                                                            |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| <p><code>ldapsearch -H ldap://&#x3C;dc> -x -b "&#x3C;domain-dn>" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength</code><br><br>The Distinguished Name (DN) for the domain follows a structure like this:<br>Domain: <code>BRM.COM</code> → DN: <code>DC=BRM,DC=COM</code></p> | **(Linux)** Retrieves password policy from the domain controller using anonymous SMB bind, if allowed. |
{% endtab %}
{% endtabs %}

***

## Authenticated

With valid domain credentials, the password policy can also be obtained remotely.

```bash
crackmapexec smb $IP -u $USER -p $PASS --pass-pol
```

### Tools

<table><thead><tr><th width="196">Name</th><th>Description</th></tr></thead><tbody><tr><td><a data-mention href="../../../toolbox/tooling/post-exploitation/crackmapexec.md">crackmapexec.md</a></td><td>An open-source tool that automates the assessment of Active Directory networks by leveraging various protocols to perform reconnaissance, credential gathering, lateral movement, and exploitation.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/information-gathering/service-enumeration/rpcclient.md">rpcclient.md</a></td><td>A command-line tool, part of the Samba suite, used to connect to and interact with Windows SMB/MS-RPC services, primarily for administration and network enumeration of users, groups, shares, and other domain information.</td></tr></tbody></table>

***

## SMB NULL Sessions

SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. SMB NULL session misconfigurations are often the result of legacy Domain Controllers (**Windows Server 2003 and earlier)** being upgraded in place, ultimately bringing along insecure configurations, which existed by default in older versions of Windows Server.

```bash
rpcclient -U "" -N $IP
```

### Tools

<table><thead><tr><th width="196">Name</th><th>Description</th></tr></thead><tbody><tr><td><a data-mention href="../../../toolbox/tooling/post-exploitation/crackmapexec.md">crackmapexec.md</a></td><td>An open-source tool that automates the assessment of Active Directory networks by leveraging various protocols to perform reconnaissance, credential gathering, lateral movement, and exploitation.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/information-gathering/service-enumeration/rpcclient.md">rpcclient.md</a></td><td>A command-line tool, part of the Samba suite, used to connect to and interact with Windows SMB/MS-RPC services, primarily for administration and network enumeration of users, groups, shares, and other domain information.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/exploitation-tools/netexec.md">netexec.md</a></td><td>The successor to CrackMapExec, designed for assessments of Active Directory environments, by performing reconnaissance, authentication testing, lateral movement, and exploitation across various protocols.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/information-gathering/linux-enumeration/enum4linux.md">enum4linux.md</a></td><td>A Perl script that wraps around various Samba tools (like <code>smbclient</code> and <code>rpcclient</code>) to automate information gathering from Windows and Samba systems, including users, groups, shares, and password policies.</td></tr></tbody></table>

***

## LDAP Anonymous Binds

[LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) allow unauthenticated attackers to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. This is a legacy configuration, and as of **Windows Server 2003**, only authenticated users are permitted to initiate LDAP requests.

### Tools

<table><thead><tr><th width="213">Name</th><th>Description</th></tr></thead><tbody><tr><td>windapsearch.py</td><td>A Python script for enumerating users, groups, computers, and other objects from Active Directory domains by querying the LDAP protocol.</td></tr><tr><td>ldapsearch</td><td>A command-line utility for querying LDAP directories, including Active Directory, to retrieve specific entries and attributes based on defined filters and scope.</td></tr><tr><td>ad-ldapdomaindump.py</td><td>A Python script that extracts detailed Active Directory information, including users, groups, computers, and policies, by querying LDAP and outputs the data into human-readable HTML, JSON, and greppable formats.</td></tr></tbody></table>

***

## Authenticated Windows Host

If we can authenticate to the domain from a Windows host, we can use built-in Windows binaries such as `net.exe` to retrieve the password policy. We can also use various tools such as PowerView, CrackMapExec ported to Windows, SharpMapExec, SharpView, etc.

### Tools

<table><thead><tr><th width="213">Name</th><th>Description</th></tr></thead><tbody><tr><td>net.exe</td><td>A built-in Windows command-line utility used for managing various network resources, including user accounts, groups, network shares, services, and connections, on both local and remote systems.</td></tr><tr><td><a data-mention href="../../../toolbox/tooling/post-exploitation/powersploit/powerview.md">powerview.md</a></td><td>A PowerShell tool for enumerating users, groups, computers, GPOs, and trusts, as well as identifying common misconfigurations and attack paths within Windows Active Directory domains.</td></tr></tbody></table>

### net.exe

```sh
C:\> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

### PowerView

```powershell
PS C:\> Import-Module .\PowerView.ps1
PS C:\> Get-DomainPolicy

Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=-1; MinimumPasswordLength=8; PasswordComplexity=1;
                 PasswordHistorySize=24; LockoutBadCount=5; ResetLockoutCount=30; LockoutDuration=30;
                 RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```
