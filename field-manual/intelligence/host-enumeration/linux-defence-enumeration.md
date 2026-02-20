# Linux Defence Enumeration

We should check to see if any defences are in place and IF we can enumerate any information about them. Some things to look for include:

* [Exec Shield](https://en.wikipedia.org/wiki/Exec_Shield)
* [iptables](https://linux.die.net/man/8/iptables)
* [AppArmor](https://apparmor.net/)
* [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
* [Fail2ban](https://github.com/fail2ban/fail2ban)
* [Snort](https://www.snort.org/faq/what-is-snort)
* [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)

Often we will not have the privileges to enumerate the configurations of these protections but knowing what, if any, are in place, can help us not to waste time on certain tasks.

***

## Cheatsheet

| Action                                                                                                            | Description                                                                                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>If the Debian <code>iptables-persistent</code> package as used:<br><code>cat /etc/iptables/rules.v4</code></p> | <p>By default, we usually need <code>root</code> access to see the Firewall setting for a system.<br><br>However, under some circumstances, we may be able to enumerate them as a low-privileged user.</p> |
