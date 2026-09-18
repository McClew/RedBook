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

# 22 - SFTP

SFTP (SSH File Transfer Protocol) is a file transfer protocol that runs over an SSH connection, distinct from the legacy plaintext FTP protocol despite the similar name. Because it rides on SSH, it inherits SSH's encryption and authentication rather than implementing its own, and it shares the same port and daemon as SSH itself.

{% hint style="info" %}
#### Default Port

22/tcp, shared with SSH. See the SSH article for service-level enumeration, banner grabbing, and algorithm auditing, since both run through the same daemon.
{% endhint %}

```
PORT   STATE SERVICE
22/tcp open  ssh
```

***

## Cheatsheet

<table><thead><tr><th width="280">Action</th><th>Description</th></tr></thead><tbody><tr><td><code>sftp &#x3C;user>@&#x3C;target></code></td><td>Connects interactively using a password.</td></tr><tr><td><code>sftp -i private.key &#x3C;user>@&#x3C;target></code></td><td>Connects using a private key.</td></tr><tr><td><code>get &#x3C;file></code></td><td>Downloads a file from the remote host.</td></tr><tr><td><code>put &#x3C;file></code></td><td>Uploads a file to the remote host.</td></tr><tr><td><code>mget</code> / <code>mput</code></td><td>Transfers multiple files, prompting for each one.</td></tr><tr><td><code>lcd</code> / <code>lpwd</code></td><td>Changes or prints the local working directory.</td></tr></tbody></table>

***

## Restricted / Chrooted SFTP

Some hosts restrict a user to SFTP-only access by setting `ForceCommand internal-sftp` (and often a `ChrootDirectory`) in `sshd_config`. In this case, valid credentials will authenticate successfully over SFTP but won't grant an interactive shell, even though the credentials themselves are entirely correct.

```bash
clue@machine[~]$ ssh -v user@10.129.14.128

<SNIP>
debug1: Entering interactive session.
debug1: pledge: filesystem
This service allows sftp connections only.
Connection to 10.129.14.128 closed.
```

{% hint style="info" %}
#### Note

A chrooted SFTP jail still allows browsing and, if writable, uploading within its own restricted directory. It's worth checking what's reachable inside the jail even without shell access, particularly for web application document roots that happen to be exposed this way.
{% endhint %}
