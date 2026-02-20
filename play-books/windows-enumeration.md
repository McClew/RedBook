---
icon: microsoft
---

# Windows Enumeration

## Internal Enumeration

\-

### Situational Awareness

Understand the immediate context and limitations.

{% stepper %}
{% step %}
#### Current User Context

Identify who we're running as, their SID and their privileges. See [users-and-groups.md](../field-manual/intelligence/windows-host-enumeration/users-and-groups.md "mention") for more information.

```bash
whoami /all
```

Check for these high-value privileges:

* `SeImpersonate`
* `SeDebug`
* `SeBackup`
{% endstep %}

{% step %}
#### Local Group Membership

Review which local groups the user belongs to, such as Event Log readers or Account Operators as these grant specific rights that can be abused. See [users-and-groups.md](../field-manual/intelligence/windows-host-enumeration/users-and-groups.md "mention") for more information.
{% endstep %}

{% step %}
#### System Information

Determine OS version, architecture and hostname. See [system.md](../field-manual/intelligence/windows-host-enumeration/system.md "mention") for more information.

```bash
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```
{% endstep %}

{% step %}
#### Defensive Posture

Check for active AV, EDR and solutions like AppLocker/Constrained Language Mode. See [defences.md](../field-manual/intelligence/windows-host-enumeration/defences.md "mention") for more information.

```powershell
Get-MpComputerStatus
```

```powershell
$ExecutionContext.SessionState.LanguageMode
```
{% endstep %}
{% endstepper %}

### System & Patch Analysis

[intelligence](../field-manual/intelligence/ "mention") > [windows-host-enumeration](../field-manual/intelligence/windows-host-enumeration/ "mention") > [system.md](../field-manual/intelligence/windows-host-enumeration/system.md "mention")

The initial phase of local enumeration involves identifying the machines configuration and security posture to find escalation paths.

{% stepper %}
{% step %}
#### System Information

Use the systeminfo command to gather basic OS details, architecture and a list of installed hotfixes.
{% endstep %}

{% step %}
#### Update Verification

To specifically target missing patches we can use:

```bash
wmic qfe list brief
```

```powershell
Get-Hotfix
```
{% endstep %}

{% step %}
#### Kernel Exploitation

Check for specific CVEs like EternalBlue (if not patched) or newer PrintNightmare / HiveNightmare variants.

#### Exploit Suggestions

Tools like `Watson` and `WES-NG` take this system information as input to identify specific missing KBs and suggest relevant privilege escalation paths.
{% endstep %}
{% endstepper %}

### Software & Process Discovery

[intelligence](../field-manual/intelligence/ "mention") > [windows-host-enumeration](../field-manual/intelligence/windows-host-enumeration/ "mention") > [software.md](../field-manual/intelligence/windows-host-enumeration/software.md "mention")

Identifying non-standard software may run with elevated privileges or contain unique vulnerabilities.

{% stepper %}
{% step %}
#### Application Inventory

Use `wmic product get name` to list installed software.
{% endstep %}

{% step %}
#### Service & Process Mapping

Map PIDs to listening ports and running services using `netstat -ano`, `get-process` and `get-service`.
{% endstep %}
{% endstepper %}

### Credential Harvesting & Log Review

[intelligence](../field-manual/intelligence/ "mention") > [windows-host-enumeration](../field-manual/intelligence/windows-host-enumeration/ "mention") > [credential-hunting.md](../field-manual/intelligence/windows-host-enumeration/credential-hunting.md "mention")

Windows hosts often store sensitive data in the registry or logs that can be extracted with local administrative access.

{% stepper %}
{% step %}
#### Automated Looting

Scan the registry and common file locations for stored credentials.

```bash
reg query HKLM /f password /t REG_SZ /s
```

Files to look for: `Unattend.xml`, `web.config`, `groups.xml` (GPP), and PowerShell history.
{% endstep %}

{% step %}
#### Manual Log Hunting

If process creation auditng is enabled, search Event ID 4688 (New Process Created) for cleartext passwords passed as command-line arguments using wevutil or Get-WinEvent.

Check for saved passwords in Chrome/Edge or the Windows Vault.
{% endstep %}
{% endstepper %}

### Advanced Lateral & Internal Networking

[intelligence](../field-manual/intelligence/ "mention") > [windows-host-enumeration](../field-manual/intelligence/windows-host-enumeration/ "mention") > [network.md](../field-manual/intelligence/windows-host-enumeration/network.md "mention")

Expand the scope to other hosts or internal services.

{% stepper %}
{% step %}
#### Internal Listeners

Identify ports listening on `127.0.0.1` that may host vulnerable management interfaces.

```bash
netstat -ano | findstr LISTENING
```
{% endstep %}

{% step %}
#### Named Pipe Permissions

Use `Pipelist.exe` or `accesschk.exe` to find named pipes with lax permissions (e.g., `FILE_ALL_ACCESS` for the "Everyone" group), which can often be leveraged for SYSTEM-level escalation.
{% endstep %}
{% endstepper %}

## Automated Enumeration

\-
