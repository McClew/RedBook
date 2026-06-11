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
---

# Linux User & Group Enumeration

## Cheatsheet

<table><thead><tr><th width="354">Action</th><th>Description</th></tr></thead><tbody><tr><td><code>cat /etc/passwd</code><br><br>Only usernames:<br><code>cut -d: -f1 /etc/passwd</code><br><br>Only username and home directories of users with shells.<br><code>awk -F: '($7 !~ /(false|nologin)$/) { print $1, $6 }' /etc/passwd</code></td><td>Lists all user accounts.<br><br>Make sure to check whether there are password hashes in <code>/etc/passwd</code>.</td></tr><tr><td>Lists all groups:<br><code>cat /etc/group</code><br><br>Only group with members:<br><code>awk -F: 'NF == 4 &#x26;&#x26; $4 != "" { print }' /etc/group</code></td><td>Lists all groups and its members.</td></tr><tr><td><code>id</code></td><td>Displays the name, UID and GID of the current user, alongside its group memberships.</td></tr><tr><td><code>env</code></td><td>Displays all shell environment variables.</td></tr><tr><td><code>who</code><br><code>w</code><br><code>finger</code></td><td>Lists currently logged in users.</td></tr><tr><td><code>lastlog</code></td><td>Lists when was the last time each user logged in.<br><br>Users who never logged in are less likely targets.</td></tr></tbody></table>

***

## Current User

### sudo Privileges

Can the user run any commands either as another user or as root? If you do not have credentials for the user, it may not be possible to leverage sudo permissions.&#x20;

{% hint style="success" %}
Often sudoer entries include `NOPASSWD`, meaning that the user can run the specified command without being prompted for a password.
{% endhint %}

Not all commands, even we can run as root, will lead to privilege escalation. It is not uncommon to gain access as a user with full sudo privileges, meaning they can run any command as root. Issuing a simple `sudo su` command will immediately give you a root session.

```bash
sudo -l
```

```bash
Matching Defaults entries for sysadm on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User sysadm may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/tcpdump
```

***

### History

It can be important to check a user's bash history, as they may be passing passwords as an argument on the command line, working with git repositories, setting up cron jobs, and more. Reviewing what the user has been doing can give you considerable insight into the type of server you land on and give a hint as to privilege escalation paths.

{% code title="Command" %}
```bash
history
```
{% endcode %}

{% code title="Example Output" %}
```bash
    1  id
    2  cd /home/cliff.moore
    3  exit
    4  touch backup.sh
    5  tail /var/log/apache2/error.log
    6  ssh ec2-user@dmz02.inlanefreight.local
    7  history
```
{% endcode %}

***

### SSH Directory

If we find an SSH key for our current user, this could be used to open an SSH session on the host (if SSH is exposed externally) and gain a stable and fully interactive session. SSH keys could be leveraged to access other systems within the network as well. At the minimum, check the ARP cache to see what other hosts are being accessed and cross-reference these against any useable SSH private keys.

{% code title="Command" %}
```bash
ls -l ~/.ssh
```
{% endcode %}

{% code title="Example Output" %}
```bash
total 8
-rw------- 1 mrb3n mrb3n 1679 Aug 30 23:37 id_rsa
-rw-r--r-- 1 mrb3n mrb3n  393 Aug 30 23:37 id_rsa.pub
```
{% endcode %}

***

### $PATH Variable

We should check out our current user's PATH, which is where the Linux system looks every time a command is executed for any executables to match the name of what we type, i.e., `id` which on this system is located at `/usr/bin/id`.

As we'll see later in this module, if the PATH variable for a target user is misconfigured we may be able to leverage it to escalate privileges. For now we'll note it down and add it to our note-taking tool of choice.

{% code title="Command" %}
```bash
echo $PATH
```
{% endcode %}

{% code title="Example Output" %}
```bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
{% endcode %}

***

### Environment Variables

We can also check out all environment variables that are set for our current user, we may get lucky and find something sensitive in there such as a password.

{% code title="Command" %}
```bash
env
```
{% endcode %}

{% code title="Example Output" %}
```bash
SHELL=/bin/bash
PWD=/home/htb-student
LOGNAME=htb-student
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/htb-student
LANG=en_US.UTF-8
<SNIP>
```
{% endcode %}

***

### SETUID & SETGID Permissions

Binaries are set with these permissions to allow a user to run a command as root, without having to grant root-level access to the user. Many binaries contain functionality that can be exploited to get a root shell.

#### 1. Using Find

The most comprehensive and standard command to search the entire file system for files with either SUID or SGID bits set is:

{% code title="Command" %}
```bash
find / -type f -perm /6000 2>/dev/null
```
{% endcode %}

`find /` Start the search from the root directory (`/`).

`-type f` Only look for files (executables), not directories.

`-perm /6000` Search for files where _ANY_ of the following bits are set:

* 4000 (SUID - Set User ID)
* 2000 (SGID - Set Group ID)
* The `/` operator acts as an OR, matching files with _either_ bit set.

`2>/dev/null` Suppress permission denied and other error messages for a cleaner output.

You can also search for the permissions individually:

Only SUID (Set User ID): `find / -type f -perm /4000 2>/dev/null4000`

Only SGID (Set Group ID): `find / -type f -perm /2000 2>/dev/null2000`

#### 2. Analysing the Output with `ls -l`

Once we have a list of files, use `ls -l` on them to confirm the special permission.

{% code title="Example SUID Output: Runs as the owner (root)" %}
```
-rwsr-xr-x 1 root root ... /bin/passwd
```
{% endcode %}

{% code title="Example SGID Output: Runs with the privileges of the group (tty)" %}
```bash
-rwxr-sr-x 1 root tty ... /usr/bin/wall
```
{% endcode %}

#### 3. Post-Enumeration Steps

The mere existence of a SUID/SGID binary is not a vulnerability, the next step is to check the binaries against a known database of exploits:

1. Check GTFObins: This is the most crucial step. Use the list of SUID/SGID binaries you found and search for them on the [GTFObins](https://gtfobins.github.io/) website. This resource documents known ways to abuse Linux binaries (often with SUID set) to gain a shell, read/write files, or bypass security restrictions.
2. Look for Suspect Files: Pay extra attention to SUID binaries owned by `root` that are not standard system utilities (like `passwd`, `su`, or `mount`). Custom-compiled programs in unusual directories are prime targets.
3. Check Path Variable/Relative Execution: Look for SUID/SGID binaries that call other programs using a relative path (e.g., calling `ls` instead of `/bin/ls`). If we can inject our own malicious version of that program into a directory that is checked earlier in the `$PATH` environment variable, we can hijack the execution and gain the elevated privileges.

***

### Writable Directories

It is important to discover which directories are writeable if we need to download tools to the system. We may discover a writeable directory where a cron job places files, which provides an idea of how often the cron job runs and could be used to elevate privileges if the script that the cron job runs is also writeable.

#### Finding Writable Directories

{% code title="Command" %}
```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
{% endcode %}

{% code title="Example Output" %}
```bash
/dmz-backups
/tmp
/tmp/VMwareDnD
/tmp/.XIM-unix
/tmp/.Test-unix
/tmp/.X11-unix
/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-TIecv0/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/proc
/dev/mqueue
/dev/shm
/var/tmp
/var/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-hm6Qdl/tmp
/var/crash
/run/lock

```
{% endcode %}

***

### Writeable Files

We scan the filesystem for files that grant write access to our current user or group. While checking for globally writable (world-writable) files across the entire system is essential, targeting critical directories like `/etc` or `/opt` often yields more immediate results.

We can look for globally writable files across the system while excluding virtual filesystems to keep our results relevant:

```bash
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

Alternatively, to focus specifically on configurations within `/etc` that our current user has permission to modify, we can use:

```bash
find /etc -writable 2>/dev/null | head -20
```

> Key Takeaway: Writable configuration files (`.conf`, `.ini`, `.sh`) must always be thoroughly analysed. If a file is writable by our user and processed or sourced by a root-run service, it represents a primary target for command injection.

***

## User Accounts

### Hunting for Users

An easy way for us to find other users is from the `/etc/passwd` file. This file is readable by all users and should provide all of the required information:

{% code title="Command" %}
```bash
cat /etc/passwd
```
{% endcode %}

{% code title="Example Output" %}
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
<...SNIP...>
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
mrb3n:x:1000:1000:mrb3n,,,:/home/mrb3n:/bin/bash
colord:x:111:118:colord colour management daemon,,,:/var/lib/colord:/bin/false
backupsvc:x:1001:1001::/home/backupsvc:
bob.jones:x:1002:1002::/home/bob.jones:
cliff.moore:x:1003:1003::/home/cliff.moore:
logger:x:1004:1004::/home/logger:
shared:x:1005:1005::/home/shared:
stacey.jenkins:x:1006:1006::/home/stacey.jenkins:
sysadm:$6$vdH7vuQIv6anIBWg$Ysk.UZzI7WxYUBYt8WRIWF0EzWlksOElDE0HLYinee38QI1A.0HW7WZCrUhZ9wwDz13bPpkTjNuRoUGYhwFE11:1007:1007::/home/sysadm:
```
{% endcode %}

***

### Login Shells

We can check which users have login shells. Once we see what shells are on the system, we can check each version for vulnerabilities. Because outdated versions, such as Bash version 4.1, are vulnerable to a `shellshock` exploit.

{% code title="Command" %}
```bash
grep "sh$" /etc/passwd
```
{% endcode %}

{% code title="Example Output" %}
```bash
root:x:0:0:root:/root:/bin/bash
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
bjones:x:1001:1001::/home/bjones:/bin/sh
administrator.ilfreight:x:1002:1002::/home/administrator.ilfreight:/bin/sh
backupsvc:x:1003:1003::/home/backupsvc:/bin/sh
cliff.moore:x:1004:1004::/home/cliff.moore:/bin/bash
logger:x:1005:1005::/home/logger:/bin/sh
shared:x:1006:1006::/home/shared:/bin/sh
stacey.jenkins:x:1007:1007::/home/stacey.jenkins:/bin/bash
htb-student:x:1008:1008::/home/htb-student:/bin/bash
```
{% endcode %}

***

### Currently Logged in Users

{% code title="Command" %}
```bash
who
```
{% endcode %}

{% code title="Example Output" %}
```bash
 12:27:21 up 1 day, 16:55,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
cliff.mo pts/0    10.10.14.16      Tue19   40:54m  0.02s  0.02s -bash
```
{% endcode %}

***

### Users Last Login

{% code title="Command" %}
```bash
lastlog
```
{% endcode %}

{% code title="Example Output" %}
```bash
Username         Port     From             Latest
root                                       **Never logged in**
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
...SNIP...
systemd-coredump                           **Never logged in**
mrb3n            pts/1    10.10.14.15      Tue Aug  2 19:33:16 +0000 2022
lxd                                        **Never logged in**
bjones                                     **Never logged in**
administrator.ilfreight                           **Never logged in**
backupsvc                                  **Never logged in**
cliff.moore      pts/0    127.0.0.1        Tue Aug  2 19:32:29 +0000 2022
logger                                     **Never logged in**
shared                                     **Never logged in**
stacey.jenkins   pts/0    10.10.14.15      Tue Aug  2 18:29:15 +0000 2022
htb-student      pts/0    10.10.14.15      Wed Aug  3 13:37:22 +0000 2022      
```
{% endcode %}

***

## Groups

### Identifying Groups

Information about the available groups can be found in the `/etc/group` file, which shows us both the group name and the assigned user names.

{% code title="Command" %}
```bash
cat /etc/group
```
{% endcode %}

{% code title="Example Output" %}
```bash
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:syslog,htb-student
tty:x:5:syslog
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:htb-student
floppy:x:25:
tape:x:26:
sudo:x:27:mrb3n,htb-student
audio:x:29:pulse
dip:x:30:htb-student
www-data:x:33:
...SNIP...

```
{% endcode %}

***

### Listing Group Members

We can use the [getent](https://man7.org/linux/man-pages/man1/getent.1.html) command to list members of any interesting groups.

{% code title="Command" %}
```bash
getent group sudo
```
{% endcode %}

{% code title="Example Output" %}
```bash
sudo:x:27:mrb3n
```
{% endcode %}

***

## Password Enumeration

### Hashing Algorithms

With Linux, several different hash algorithms can be used to secure passwords. Identifying them from the first hash blocks can help us to use and work with them later if needed. Here is a list of the most used ones:

* Salted MD5 `$1$...`
* SHA-256 `$5$...`
* SHA-512 `$6$...`
* BCrypt `$2a$...`
* Scrypt `$7$...`
* Argon2 `$argon2i$...`

***

### /etc/passwd

Occasionally, we may see password hashes directly in the `/etc/passwd` file. This file is readable by all users, and as with hashes in the `shadow` file, these can be subjected to an offline password cracking attack. This configuration, while not common, can sometimes be seen on embedded devices and routers.

***

### /etc/shadow

If the shadow file is readable, we will be able to gather password hashes for all users who have a password set. While this does not guarantee further access, these hashes can be subjected to an offline brute-force attack to recover the cleartext password.
