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
  anchors:
    visible: true
---

# Linux System Enumeration

## Cheatsheet

{% tabs %}
{% tab title="Operating System" %}
<table><thead><tr><th width="326">Action</th><th>Description</th></tr></thead><tbody><tr><td><code>hostname</code></td><td>Displays hostname of system.<br><br>This may hint at the purpose of the host.</td></tr><tr><td><p>Linux distro:<br><code>cat /etc/issue</code><br><code>cat /etc/*-release</code><br><br>Kernel version .<br><code>(cat /proc/version || uname -a )</code> </p><p><code>2>/dev/null</code></p></td><td>Displays the Linux distribution and kernel version.</td></tr><tr><td><code>arch</code></td><td>Displays the CPU architecture.</td></tr><tr><td><code>realm list</code></td><td>Check if Linux host is Active Directory domain-joined.<br><br>If it is, this command will output a bunch of information. Pay attention to the fields:<br>- <code>domain-name</code><br>- <code>permitted-logins</code><br>- <code>permitted-groups</code><br>- <code>configured</code> (if it’s <code>kerberos-member</code>, PtH is possible)</td></tr><tr><td><code>ps -ef | grep -i "winbind|sssd"</code></td><td>If the realms command does not exist, try to footprint by searching for AD tools such as <code>winbind</code> and <code>sssd</code>.</td></tr></tbody></table>
{% endtab %}

{% tab title="File System" %}
| Action                                                                              | Description                                                              |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `ls -la /opt /var/backups /var/opt /var/mail /tmp /var/tmp`                         | Check the contents of directories that often contains interesting files. |
| `find / -writable -type d 2>/dev/null`                                              | Finds writable directories by the current user.                          |
| `find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;` | Finds files with capabilities set in common system binary directories.   |
{% endtab %}
{% endtabs %}

***

## Tools

Several helper scripts such as [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) and [LinEnum](https://github.com/rebootuser/LinEnum) exist to assist with enumeration.

***

## Unix Name

This command provides a comprehensive overview of your system's kernel and hardware, including:

* Kernel name: (e.g., Linux)
* Network node hostname: The name of your machine on the network.
* Kernel release: The specific version of your kernel (e.g., 5.15.0-76-generic).
* Kernel version: More detailed information about the kernel build, including the compile date and compiler used.
* Machine hardware name: The architecture of your hardware (e.g., x86\_64).
* Processor type: (e.g., x86\_64)
* Hardware platform: (e.g., x86\_64)
* Operating system: (e.g., GNU/Linux)

{% code title="Command" %}
```bash
uname -a
```
{% endcode %}

`uname`: This command (short for "Unix name") prints system information.

`-a`: This option tells `uname` to print all available information.

{% code title="Example Output" overflow="wrap" %}
```bash
Linux my-machine 5.15.0-76-generic #83-Ubuntu SMP Mon Jun 19 16:32:04 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```
{% endcode %}

***

## Proc File System

This file contains a string that identifies the version of the Linux kernel currently running, including the GCC version used to compile it and the time of compilation. It's often very similar to the "Kernel version" part of `uname -a`.

{% code title="Command" %}
```bash
cat /proc/version
```
{% endcode %}

`/proc/version`: This is a virtual file within the `/proc` filesystem. The `/proc` filesystem is a pseudo-filesystem that provides an interface to kernel data structures.

{% code title="Example Output" overflow="wrap" %}
```bash
Linux version 5.15.0-76-generic (buildd@lcy02-amd64-060) (gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #83-Ubuntu SMP Mon Jun 19 16:32:04 UTC 2023
```
{% endcode %}

***

## Operating System Identification

This file typically displays the operating system's name and version, and sometimes additional information like the architecture. It's a quick way to see what Linux distribution is running.

{% code title="Command" %}
```bash
cat /etc/issue
```
{% endcode %}

`/etc/issue`: This file contains a system identification string that is displayed _before_ the login prompt. It's often used by distributions to show their name and version.

{% code title="Ubuntu Output" %}
```bash
Ubuntu 22.04.2 LTS \n \l
```
{% endcode %}

<pre class="language-bash" data-title="CentOS/RHEL Output"><code class="lang-bash"><strong>CentOS Linux release 8.5.2111 (Core)
</strong>Kernel \r on an \m
</code></pre>

***

## Login Shells

{% code title="Command" %}
```bash
cat /etc/shells
```
{% endcode %}

{% code title="Example Output" %}
```bash
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
```
{% endcode %}

***

## File System & Unmounted Drives

### Discovering Drives

If we discover - and can mount - an additional drive or unmounted file system, we may find sensitive files, passwords, or backups that can be leveraged to escalate privileges.

{% code title="Command" %}
```bash
lsblk
```
{% endcode %}

{% code title="Example Output" %}
```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   30G  0 disk 
├─sda1   8:1    0   29G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1  848M  0 rom  
```
{% endcode %}

### Protected Drive Passwords

`/etc/fstab` may contain credentials for protect drives. We can attempt to find these credentials - or where they're stored by grepping for common words such as password, username, credential, etc in `/etc/fstab`.

{% code title="Command" %}
```bash
cat /etc/fstab | grep -E 'credential|password|username'
```
{% endcode %}

{% code title="Example Output: Network share with credentials (common)" %}
```bash
//192.168.1.10/share /mnt/share cifs credentials=/home/user/.smbcreds 0 0
```
{% endcode %}

{% code title="Example Output: Credentials stored directly (uncommon but possible)" %}
```bash
//192.168.1.11/backup /mnt/backup cifs rw,user,username=backupuser,password=secretpass 0 0
```
{% endcode %}

***

### Finding Hidden Files

{% code title="Command" %}
```bash
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep rhys
```
{% endcode %}

{% code title="Example Output" %}
```bash
-rw-r--r-- 1 rhys rhys 3771 Nov 27 11:16 /home/rhys/.bashrc
-rw-rw-r-- 1 rhys rhys 180 Nov 27 11:36 /home/rhys/.wget-hsts
-rw------- 1 rhys rhys 387 Nov 27 14:02 /home/rhys/.bash_history
-rw-r--r-- 1 rhys rhys 807 Nov 27 11:16 /home/rhys/.profile
-rw-r--r-- 1 rhys rhys 0 Nov 27 11:31 /home/rhys/.sudo_as_admin_successful
-rw-r--r-- 1 rhys rhys 220 Nov 27 11:16 /home/rhys/.bash_logout
-rw-rw-r-- 1 rhys rhys 162 Nov 28 13:26 /home/rhys/.notes
```
{% endcode %}

***

### Finding Hidden Directories

{% code title="Command" %}
```bash
find / -type d -name ".*" -ls 2>/dev/null
```
{% endcode %}

{% code title="Example Output" %}
```bash
   684822      4 drwx------   3 rhys rhys                   4096 Nov 28 12:32 /home/rhys/.gnupg
   790793      4 drwx------   2 rhys rhys                   4096 Okt 27 11:31 /home/rhys/.ssh
   684804      4 drwx------  10 rhys rhys                   4096 Okt 27 11:30 /home/rhys/.cache
   790827      4 drwxrwxr-x   8 rhys rhys                   4096 Okt 27 11:32 /home/rhys/CVE-2021-3156/.git
   684796      4 drwx------  10 rhys rhys                   4096 Okt 27 11:30 /home/rhys/.config
   655426      4 drwxr-xr-x   3 rhys rhys                   4096 Okt 27 11:19 /home/rhys/.local
   524808      4 drwxr-xr-x   7 gdm         gdm             4096 Okt 27 11:19 /var/lib/gdm3/.cache
   544027      4 drwxr-xr-x   7 gdm         gdm             4096 Okt 27 11:19 /var/lib/gdm3/.config
   544028      4 drwxr-xr-x   3 gdm         gdm             4096 Aug 31 08:54 /var/lib/gdm3/.local
   524938      4 drwx------   2 colord      colord          4096 Okt 27 11:19 /var/lib/colord/.cache
     1408      2 dr-xr-xr-x   1 rhys rhys                   2048 Aug 31 09:17 /media/rhys/Ubuntu\ 20.04.5\ LTS\ amd64/.disk
   280101      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.font-unix
   262364      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:32 /tmp/.ICE-unix
   262362      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:32 /tmp/.X11-unix
   280103      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.Test-unix
   262830      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.XIM-unix
   661820      4 drwxr-xr-x   5 root        root            4096 Aug 31 08:55 /usr/lib/modules/5.15.0-46-generic/vdso/.build-id
   666709      4 drwxr-xr-x   5 root        root            4096 Okt 27 11:18 /usr/lib/modules/5.15.0-52-generic/vdso/.build-id
   657527      4 drwxr-xr-x 170 root        root            4096 Aug 31 08:55 /usr/lib/debug/.build-id
```
{% endcode %}

***

### Temporary Files

There are three default folders are intended for temporary files. These folders are visible to all users and can be read. In addition, temporary logs or script output can be found there.

Both `/tmp` and `/var/tmp` are used to store data temporarily. However, the key difference is how long the data is stored in these file systems. The data retention time for `/var/tmp` is much longer than that of the `/tmp` directory.

{% hint style="info" %}
By default, all files and data stored in **/var/tmp** are retained for **up to 30 days**. In **/tmp**, on the other hand, the data is **automatically deleted after ten days**.
{% endhint %}

Additionally all temporary files stored in the `/tmp` directory are deleted immediately when the system is restarted. Therefore, the `/var/tmp` directory is used by programs to store data that must be kept between reboots temporarily.

{% code title="Command" %}
```bash
ls -l /tmp /var/tmp /dev/shm
```
{% endcode %}

{% code title="Example Output" %}
```bash
/dev/shm:
total 0

/tmp:
total 52
-rw------- 1 htb-student htb-student    0 Nov 28 12:32 config-err-v8LfEU
drwx------ 3 root        root        4096 Nov 28 12:37 snap.snap-store
drwx------ 2 htb-student htb-student 4096 Nov 28 12:32 ssh-OKlLKjlc98xh
<SNIP>
drwx------ 2 htb-student htb-student 4096 Nov 28 12:37 tracker-extract-files.1000
drwx------ 2 gdm         gdm         4096 Nov 28 12:31 tracker-extract-files.125

/var/tmp:
total 28
drwx------ 3 root root 4096 Nov 28 12:31 systemd-private-7b455e62ec09484b87eff41023c4ca53-colord.service-RrPcyi
drwx------ 3 root root 4096 Nov 28 12:31 systemd-private-7b455e62ec09484b87eff41023c4ca53-ModemManager.service-4Rej9e
...SNIP...
```
{% endcode %}

***

## Discover Printers

The command `lpstat` can be used to find information about any printers attached to the system. If there are active or queued print jobs we can attempt to gain access and maybe find some sensitive information.

{% code title="Command" %}
```bash
lpstat
```
{% endcode %}

{% code title="Example Output (if jobs are queued)" %}
```bash
LaserJet_P2-14  jdoe  81920   Sep 26 10:35
PDF_Printer-15  jdoe  15360   Sep 26 12:05
```
{% endcode %}

Use the below command to get a more verbose output:

```
lpstat -t
```

This will show all status information including:

* Default printer destination
* Status of all printers
* CUPS scheduler status
* Status summary
