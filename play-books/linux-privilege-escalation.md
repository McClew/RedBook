---
icon: linux
---

# Linux Privilege Escalation

### Sudo & Capability Exploitation

Investigate the permissions granted to the current user that allow for execution as other users or with elevated system privileges.

{% stepper %}
{% step %}
#### Sudo Rights

Run `sudo -l` to list allowed commands and look for `(root) NOPASSWD` entries.
{% endstep %}

{% step %}
#### GTFOBins Mapping

For any binary listed in `sudo -l`, check [GTFOBins](https://gtfobins.github.io/) for shell escape sequences.
{% endstep %}

{% step %}
#### LD\_PRELOAD Analysis

If `env_keep += LD_PRELOAD` is present in sudo settings, attempt shared object hijacking to gain root.
{% endstep %}

{% step %}
#### Linux Capabilities

Use `getcap -r / 2>/dev/null` to find binaries with special permissions like `cap_setuid` or `cap_dac_override`.
{% endstep %}
{% endstepper %}

### SUID/SGID and Binary Hijacking

Focus on binaries that execute with the owner's privileges rather than the user's.

{% stepper %}
{% step %}
#### SUID/SGID Discovery

Locate all SUID files using `find / -perm -u=s -type f 2>/dev/null`.
{% endstep %}

{% step %}
#### Custom Binary Analysis

If a custom (non-standard) SUID binary exists, use `strings` or `ltrace` to find calls to external programs or shared objects.
{% endstep %}

{% step %}
#### Shared Object (SO) Hacking

Identify if an SUID binary is looking for a `.so` file in a writable directory and provide a malicious version.
{% endstep %}

{% step %}
#### Path Hijacking

If a privileged script/binary calls a program without using an absolute path, attempt to hijack the `$PATH` variable.
{% endstep %}
{% endstepper %}

### Scheduled Tasks and Service Abuse

Identify background processes or system services that can be manipulated.

{% stepper %}
{% step %}
#### Cron Job Analysis

Check `/etc/crontab` and `/etc/cron.d/` for jobs running as root that execute writable scripts.
{% endstep %}

{% step %}
#### Wildcard Abuse

Look for `tar`, `chown`, or `rsync` commands in cron jobs that use wildcards (`*`), which can be exploited for command injection.
{% endstep %}

{% step %}
#### Logrotate Exploitation

Check for vulnerable versions of `logrotate` that may allow for a race condition to replace a log file with a symlink to a sensitive system file.
{% endstep %}

{% step %}
#### Timer and Service Files

Check systemd timer files (`.timer`) and service files (`.service`) for insecure permissions.
{% endstep %}
{% endstepper %}

### Application and Credentials Deep-Dive

Search for sensitive data stored within application-specific files that lead to credential reuse or direct exploitation.

{% stepper %}
{% step %}
#### Web Configuration Audit

Manually inspect `/var/www/html/`, `/srv/http/`, or `/opt/` for database connection strings (e.g., `wp-config.php`, `settings.py`, `.env`).
{% endstep %}

{% step %}
#### Credential Hunting

Search the file system for "password", "pwd", or "secret" within configuration files and source code.
{% endstep %}

{% step %}
#### SSH Key Harvesting

Search all user home directories for `.ssh/id_rsa` or `.ssh/authorized_keys`.
{% endstep %}

{% step %}
#### Internal Traffic Capture

If `tcpdump` is available with appropriate permissions, sniff local traffic for cleartext credentials (HTTP, FTP, Telnet).
{% endstep %}
{% endstepper %}

### Kernel and Software Vulnerabilities

If configuration-based escalation is not possible, look for vulnerabilities in the OS kernel or installed software versions.

{% stepper %}
{% step %}
#### Kernel Exploits

Cross-reference the output of `uname -a` with exploit databases for LPEs like _DirtyPipe_, _PwnKit_, or _CVE-2021-3493_.
{% endstep %}

{% step %}
#### Python Library Hijacking

If a script running as root imports a module, check if you have write access to the directory containing that module.
{% endstep %}

{% step %}
#### Exploit Prototyping

Always verify kernel exploits in a local lab environment before running them on a target to avoid crashing the system.
{% endstep %}
{% endstepper %}
