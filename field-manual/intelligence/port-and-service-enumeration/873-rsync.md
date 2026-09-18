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

# 873 - Rsync

**Rsync** synchronises files and directories efficiently by transferring only the differences between source and destination. Run as a standalone daemon (`rsyncd`), it exposes one or more named **modules** - each mapping to a directory tree on the server - which clients connect to directly, without needing an SSH account.

{% hint style="info" %}
The Rsync daemon listens on **TCP port 873**. This is distinct from the far more common use of `rsync` tunnelled over SSH, which uses port 22 instead and inherits SSH's authentication.&#x20;
{% endhint %}

```
PORT    STATE SERVICE
873/tcp open  rsync
```

***

## Cheat Sheet

| Command                                                  | Description                                                       |
| -------------------------------------------------------- | ----------------------------------------------------------------- |
| `nmap -sV -p873 --script rsync-list-modules <TARGET_IP>` | Lists all published modules without needing the `rsync` client.   |
| `rsync -av --list-only rsync://<TARGET_IP>/`             | Lists modules using the `rsync` client directly.                  |
| `rsync -av --list-only rsync://<TARGET_IP>/<MODULE>/`    | Lists the contents of a specific module.                          |
| `rsync -av rsync://<TARGET_IP>/<MODULE>/ ./loot/`        | Recursively **downloads** the entire module to a local directory. |
| `rsync -av <LOCAL_FILE> rsync://<TARGET_IP>/<MODULE>/`   | **Uploads** a file into the module, if it is writable.            |

***

## Anonymous / Misconfigured Access

Modules can be configured with `auth users` and a secrets file, but very commonly are left with **no authentication** at all, or a writable module intended to be read-only. Always check both directions:

```bash
# Enumerate every module first
rsync -av --list-only rsync://<TARGET_IP>/

# Then, for each module, check what it exposes and whether you can write to it
rsync -av --list-only rsync://<TARGET_IP>/<MODULE>/
echo "test" > /tmp/rsynctest && rsync -av /tmp/rsynctest rsync://<TARGET_IP>/<MODULE>/
```

{% hint style="warning" %}
A **writable, anonymous module** that maps to a sensitive location (a webroot, a cron-executed directory, an SSH `authorized_keys` path, or a home directory) is a direct route to remote code execution or persistence - treat it the same way you would an anonymous, writable FTP or SMB share.
{% endhint %}

If credentials are required, `rsync` will prompt interactively; supply them non-interactively with `--password-file` or the `RSYNC_PASSWORD` environment variable once known.

***

## Automated

```bash
nmap -sV -p873 --script rsync-list-modules <TARGET_IP>
```
