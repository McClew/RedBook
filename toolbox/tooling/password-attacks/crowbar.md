# Crowbar

Crowbar is a brute forcing tool that was developed to brute force some protocols in a different manner according to other popular brute forcing tools. As an example, while most brute forcing tools use username and password for SSH brute force, Crowbar uses SSH key(s). This allows for any private keys that have been obtained during penetration tests, to be used to attack other SSH servers.

Crowbar supports:

* OpenVPN (`-b openvpn`)
* Remote Desktop Protocol (RDP) with NLA support (`-b rdp`)
* SSH private key authentication (`-b sshkey`)
* VNC key authentication (`-b vpn`)

{% hint style="info" %}
## Download & Install:

Download from GitHub:

[https://github.com/galkan/crowbar](https://github.com/galkan/crowbar)
{% endhint %}
