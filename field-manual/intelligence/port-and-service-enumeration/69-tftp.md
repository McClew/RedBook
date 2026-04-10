---
icon: ethernet
---

# 69 - TFTP

Trivial File Transfer Protocol (TFTP) is simpler than FTP and performs file transfers between client and server processes. However, it **does not** provide user authentication and other valuable features supported by FTP. In addition, while FTP uses TCP, TFTP uses `UDP`, making it an unreliable protocol and causing it to use UDP-assisted application layer recovery.

It does not support protected login via passwords and sets limits on access based solely on the read and write permissions of a file in the operating system. Because of the lack of security, TFTP, unlike FTP, may only be used in local and protected networks.

***

## Cheatsheet

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits TFTP.                                                                                                                            |
| `status`     | Shows the current status of TFTP, including the current transfer mode (ASCII or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |

Unlike the FTP client, TFTP does not have directory listing functionality.

***

## Basic Use

{% code title="Get File" %}
```bash
tftp -i <TARGET_IP> get <FILE>
```
{% endcode %}

***

## Default Directory

The default system folder for TFTP file storage on Linux systems is generally `/var/lib/tftpboot/`. While this is the standard location for servers like `tftpd-hpa` on Ubuntu/Debian, other systems might use `/tftpboot/` or `/etc/tftpboot/` depending on the configuration.
