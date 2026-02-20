# Linux Credential Hunting

Credentials may be found in configuration files (`.conf`, `.config`, `.xml`, etc.), shell scripts, a user's bash history file, backup (`.bak`) files, within database files or even in text files. Credentials may be useful for escalating to other users or even root, accessing databases and other systems within the environment.

## Login Credentials

### Hashing Algorithms <a href="#hashing-algorithms" id="hashing-algorithms"></a>

With Linux, several different hash algorithms can be used to secure passwords. Identifying them from the first hash blocks can help us to use and work with them later if needed. Here is a list of the most used ones:

* Salted MD5 `$1$...`
* SHA-256 `$5$...`
* SHA-512 `$6$...`
* BCrypt `$2a$...`
* Scrypt `$7$...`
* Argon2 `$argon2i$...`

***

### /etc/passwd <a href="#etc-passwd" id="etc-passwd"></a>

Occasionally, we may see password hashes directly in the `/etc/passwd` file. This file is readable by all users, and as with hashes in the `shadow` file, these can be subjected to an offline password cracking attack. This configuration, while not common, can sometimes be seen on embedded devices and routers.

***

### /etc/shadow <a href="#etc-shadow" id="etc-shadow"></a>

If the shadow file is readable, we will be able to gather password hashes for all users who have a password set. While this does not guarantee further access, these hashes can be subjected to an offline brute-force attack to recover the cleartext password.

***

## Configuration Enumeration

The /var directory typically contains the web root for whatever web server is running on the host. The web root may contain database credentials or other types of credentials that can be leveraged to further access. A common example is MySQL database credentials within WordPress configuration files:

{% code title="Command" %}
```bash
grep 'DB_USER\|DB_PASSWORD' wp-config.php
```
{% endcode %}

{% code title="Example Output" %}
```bash
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'WPadmin123!' );
```
{% endcode %}

***

## Mail Directory Enumeration

The spool or mail directories, if accessible, might contain valuable information or even credentials. It is common to find credentials stored in files in the web root (i.e. MySQL connection strings, WordPress configuration files).

{% code title="Command" %}
```bash
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```
{% endcode %}

{% code title="Example Output" %}
```bash
/etc/ssh/ssh_config
/etc/ssh/sshd_config
/etc/python3/debian_config
/etc/kbd/config
/etc/manpath.config
/boot/config-4.4.0-116-generic
/boot/grub/i386-pc/configfile.mod
/sys/devices/pci0000:00/0000:00:00.0/config
/sys/devices/pci0000:00/0000:00:01.0/config
<SNIP>
```
{% endcode %}

***

## SSH Keys

&#x20;We may locate a private key for another, more privileged, user that we can use to connect back to the box with additional privileges. We may also sometimes find SSH keys that can be used to access other hosts in the environment.

Whenever finding SSH keys check the `known_hosts` file to find targets. This file contains a list of public keys for all the hosts which the user has connected to in the past and may be useful for lateral movement or to find data on a remote host that can be used to perform privilege escalation on our target.
