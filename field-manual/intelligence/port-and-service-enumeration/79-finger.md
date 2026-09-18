---
icon: ethernet
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

# 79 - Finger

The **Finger protocol** (RFC 1288) was designed to let a remote user query information about accounts on a Unix host: full name, home directory, shell, whether they are currently logged in, from where, and how long they have been idle.

{% hint style="info" %}
Finger listens on **TCP port 79**. It predates any concept of access control on the information it returns, which is exactly why it is almost never enabled on modern systems outside legacy environments and CTF boxes.
{% endhint %}

```
PORT   STATE SERVICE
79/tcp open  finger
```

***

## Cheatsheet

| Command                              | Description                                                                                                 |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `finger @<TARGET_IP>`                | Lists all users currently logged in on the target.                                                          |
| `finger <USER>@<TARGET_IP>`          | Returns detailed information for a specific username.                                                       |
| `finger <USER>@@<TARGET_IP>`         | Some implementations forward the query, useful for pivoting through a Finger gateway.                       |
| `echo "<USER>" \| nc <TARGET_IP> 79` | Raw protocol query if no `finger` client is installed — the server expects the username followed by `\r\n`. |

***

## Username Enumeration

Finger is most valuable during recon as a **username oracle**. Many implementations behave differently for valid vs. invalid usernames (different response text, or an error for unknown users vs. account details for known ones), which makes it well suited to enumeration:

```bash
for user in $(cat users.txt); do
  echo -n "$user: "
  finger "$user"@<TARGET_IP> 2>/dev/null | head -1
done
```

Confirmed usernames feed directly into brute-force attacks against SSH, FTP, SMB, or any other exposed authentication service on the same host.

***

## Automated

```bash
nmap -sV -p79 --script finger <TARGET_IP>
```
