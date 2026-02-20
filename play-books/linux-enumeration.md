---
icon: linux
---

# Linux Enumeration

## Internal Enumeration

### System & Kernel

[intelligence](../field-manual/intelligence/ "mention") > [host-enumeration](../field-manual/intelligence/host-enumeration/ "mention") > [linux-system-enumeration.md](../field-manual/intelligence/host-enumeration/linux-system-enumeration.md "mention")

{% stepper %}
{% step %}
#### System Identification

Gather the basic operating system details, distribution version, and kernel release to identify known vulnerabilities (e.g., DirtyPipe, PwnKit).

**Commands:**

```bash
uname -a
```

```bash
cat /etc/os-release
```

```bash
hostnamectl
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### Kernel Exploitation Check

Compare the kernel version against exploit databases like [searchsploit.md](../toolbox/tooling/exploitation-tools/searchsploit.md "mention") or `Exploit-DB`. Identify if the kernel is unpatched against public local privilege escalation (LPE) exploits.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Environment Variables

Review the `$PATH` for writable directories and check for sensitive information (API keys, secrets) stored in environment variables.

**Command:**

```bash
env
```

```bash
printenv
```

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### User & Group

[intelligence](../field-manual/intelligence/ "mention") > [host-enumeration](../field-manual/intelligence/host-enumeration/ "mention") > [linux-user-and-group-enumeration.md](../field-manual/intelligence/host-enumeration/linux-user-and-group-enumeration.md "mention")

{% stepper %}
{% step %}
#### Current User Context

Determine our current privileges, group memberships, and special capabilities.

**Commands:**

```bash
id
```

```bash
whoami
```

```bash
getcap -r / 2>/dev/null
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### User List & Login History

Identify other human users on the system and see who has logged in recently to find high-value targets.

**Commands:**&#x20;

```bash
cat /etc/passwd | cut -d: -f1
```

```bash
last
```

```bash
w
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### Sudo Permissions

Check if the current user can execute commands as root or another user, specifically looking for `NOPASSWD` entries.

**Commands:**

```bash
sudo -l
```

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### Network

[intelligence](../field-manual/intelligence/ "mention") > [host-enumeration](../field-manual/intelligence/host-enumeration/ "mention") > [linux-network-enumeration.md](../field-manual/intelligence/host-enumeration/linux-network-enumeration.md "mention")

{% stepper %}
{% step %}
#### Internal Network Connections

Identify services running locally (127.0.0.1) that were not visible during your external Nmap scan.

**Commands:**

```bash
netstat -ano
```

```bash
ss -tulnp
```

* [ ] Complete

***
{% endstep %}

{% step %}
### Routing & Arp Tables

Determine if the host is multi-homed (connected to multiple networks), which could allow for pivoting.

**Commands:**

```bash
ip route
```

```bash
arp -a
```

* [ ] Complete

***
{% endstep %}

{% step %}
### Active Internal Traffic

Monitor internal traffic briefly to identify active communication between the host and other internal assets.

**Commands:**

```bash
tcpdump -i eth0 -c 50
```

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### Programs & Processes

[intelligence](../field-manual/intelligence/ "mention") > [host-enumeration](../field-manual/intelligence/host-enumeration/ "mention") > [linux-software-enumeration.md](../field-manual/intelligence/host-enumeration/linux-software-enumeration.md "mention")

{% stepper %}
{% step %}
#### Running Processes & Services

Look for processes running as `root` or other users, especially custom binaries or non-standard services.

**Commands:**

```bash
ps aux
```

```bash
top
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### Cron Jobs & Scheduled Tasks

Examine system-wide and user-specific crontabs for scripts that run with elevated privileges.

**Commands:**

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.*
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### Installed Software & Package Versions

List installed packages to check for outdated versions with known vulnerabilities.

**Commands:**

{% code title="Debian / Ubuntu" %}
```bash
dpkg -l
```
{% endcode %}

{% code title="RHEL / CentOS" %}
```bash
rpm -qa
```
{% endcode %}

* [ ] Complete

***
{% endstep %}

{% step %}
#### Targeted Program Hunting

If a specific application (e.g., OsTicket, WordPress, CMS) is identified, pivot to locating its configuration and source files.

**Locate Web Files:**

Check common web roots like `/var/www/html/` or `/opt/`.

**Hunt for Configs:**

Search for keywords like `config.php`, `settings.py`, `.env`, or `web.config`.

**Credential Extraction:**

Audit these files for database passwords, API keys, or hardcoded administrative credentials.

**Commands:**

```bash
find / -name "*config*" 2>/dev/null
```

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### File System & Sensitive Data

[intelligence](../field-manual/intelligence/ "mention") > [host-enumeration](../field-manual/intelligence/host-enumeration/ "mention") > [linux-credential-hunting.md](../field-manual/intelligence/host-enumeration/linux-credential-hunting.md "mention")

{% stepper %}
{% step %}
#### SUID/SGID Binaries

Locate files with the SUID bit set, which execute with the privileges of the file owner (often root).

**Commands:**

```bash
find / -perm -u=s -type f 2>/dev/null
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### Writable Files & Directories

Search for configuration files or system scripts that the current user has write access to.

**Commands:**

```bash
find / -writable -type f 2>/dev/null | grep -v "/proc"
```

* [ ] Complete

***
{% endstep %}

{% step %}
#### SSH Key & Config Harvesting

Check home directories for hidden SSH keys, known\_hosts, or configuration files containing credentials.

**Commands:**

```bash
ls -la ~/.ssh/
```

```bash
cat /etc/ssh/sshd_config
```

* [ ] Complete

***
{% endstep %}
{% endstepper %}

***

## Automated Enumeration

Use automated tools to map the attack surface. This saves time and identifies common misconfigurations instantly.

### Initial Automated Auditing

{% stepper %}
{% step %}
#### Automated Script

Run `LinPeas` or `lse.sh` (Linux Smart Enumeration).

* [ ] Complete

***
{% endstep %}

{% step %}
#### Output Management

Redirect tool output to a file and transfer it to your attack host for persistent analysis using `grep` or automated parsers.

* [ ] Complete

***
{% endstep %}
{% endstepper %}
