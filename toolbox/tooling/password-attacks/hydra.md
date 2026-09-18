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

# Hydra

{% hint style="info" %}
#### Download & Install:

{% embed url="https://github.com/vanhauser-thc/thc-hydra" %}

Install:

```bash
sudo apt update && sudo apt install -y hydra
```
{% endhint %}

**Hydra** (THC-Hydra) is a fast, parallelised login brute-forcer supporting a very wide range of protocols and services - SSH, FTP, RDP, SMB, HTTP forms, databases, mail protocols, VNC, and many more. It's the standard tool for credential brute-forcing/spraying against a single service once a target and protocol are known.

#### Practical Notes

* <mark style="color:$primary;">**`-e nsr`**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**catches easy wins cheaply.**</mark> Null passwords, password==username, and reversed username are common enough in weak environments that it's worth always including this rather than relying purely on wordlists.
* <mark style="color:$primary;">**`S=`**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**beats**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**`F=`**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**where possible.**</mark> Success-string matching is generally more reliable than failure-string matching for HTTP form modules, since failure pages sometimes share text with other unrelated pages on the same site.
* <mark style="color:$primary;">**Combo files (**</mark><mark style="color:$primary;">**`-C`**</mark><mark style="color:$primary;">**) suit leaked-credential reuse testing.**</mark> When testing whether previously breached username:password pairs still work, `-C` is more natural than maintaining separate user/password lists.
* <mark style="color:$primary;">**Verify hits manually.**</mark> As with any automated brute-force tool, always manually confirm a reported valid credential actually authenticates before reporting it — timing artefacts and inconsistent error pages can occasionally produce false positives, particularly on HTTP form modules.

***

## Basic Syntax

```bash
hydra -l <username> -P <wordlist> <target> <service>
```

Or using URI-style targets, which Hydra also accepts directly:

```bash
hydra -l <username> -P <wordlist> <service>://<target>
```

***

## Specifying Credentials

<table><thead><tr><th width="109.6666259765625">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-l</code></td><td>Single username</td></tr><tr><td><code>-L</code></td><td>File containing a list of usernames</td></tr><tr><td><code>-p</code></td><td>Single password</td></tr><tr><td><code>-P</code></td><td>File containing a list of passwords</td></tr><tr><td><code>-C</code></td><td>Combo file, <code>username:password</code> per line (replaces <code>-l</code>/<code>-L</code> and <code>-p</code>/<code>-P</code>)</td></tr><tr><td><code>-e nsr</code></td><td>Also try: <code>n</code> = null password, <code>s</code> = same as username, <code>r</code> = reversed username</td></tr></tbody></table>

***

### Protocol Examples

#### SSH

```bash
hydra -l root -P /opt/useful/seclists/Passwords/darkweb2017-top10000.txt ssh://SERVER_IP
```

#### FTP

```bash
hydra -L users.txt -P passwords.txt ftp://SERVER_IP
```

#### RDP

```bash
hydra -l administrator -P passwords.txt rdp://SERVER_IP
```

#### SMB

```bash
hydra -L users.txt -P passwords.txt smb://SERVER_IP
```

{% hint style="info" %}
#### Best Tool for the Job

For SMB/AD spraying against multiple hosts or accounts at scale, [netexec.md](../exploitation-tools/netexec.md "mention") (formerly [crackmapexec.md](../post-exploitation/crackmapexec.md "mention")) is generally the better tool. Hydra is best suited to single-service, single-target brute-forcing.
{% endhint %}

#### MySQL / MSSQL

```bash
hydra -l root -P passwords.txt mysql://SERVER_IP
hydra -l sa -P passwords.txt mssql://SERVER_IP
```

#### Telnet

```bash
hydra -l admin -P passwords.txt telnet://SERVER_IP
```

#### POP3 / IMAP

```bash
hydra -l user@inlanefreight.com -P passwords.txt pop3://SERVER_IP
hydra -l user@inlanefreight.com -P passwords.txt imap://SERVER_IP
```

#### HTTP Basic Auth

```bash
hydra -l admin -P passwords.txt SERVER_IP http-get /admin/
```

#### HTTP POST Form Login

This is the fiddliest one — the module string needs the login path, the POST body with `^USER^`/`^PASS^` placeholders, and a failure condition:

{% code overflow="wrap" %}
```bash
hydra -l admin -P passwords.txt SERVER_IP http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid credentials"
```
{% endcode %}

{% hint style="info" %}
#### String Matching

The `F=` string must be text that appears **only** on a failed login (e.g. an error message). Alternatively, use `S=` to match a string that appears only on **success** (e.g. a redirect location or "Welcome" banner) - often more reliable than matching on failure text.
{% endhint %}

#### HTTPS Form Login

Same as above, just swap the module name:

{% code overflow="wrap" %}
```bash
hydra -l admin -P passwords.txt SERVER_IP https-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid credentials"
```
{% endcode %}

#### VNC

```bash
hydra -P passwords.txt vnc://SERVER_IP
```

***

## Performance & Behaviour Flags

<table><thead><tr><th width="98.33331298828125">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-t</code></td><td>Number of parallel tasks/connections (default: 16)</td></tr><tr><td><code>-f</code></td><td>Stop as soon as one valid login/password pair is found (single target)</td></tr><tr><td><code>-F</code></td><td>Same as <code>-f</code>, but across all targets when brute-forcing multiple hosts</td></tr><tr><td><code>-s</code></td><td>Specify a non-default port</td></tr><tr><td><code>-M</code></td><td>File containing a list of target hosts (for the same service across many targets)</td></tr><tr><td><code>-o</code></td><td>Write found credentials to a file</td></tr><tr><td><code>-v</code> / <code>-V</code></td><td>Verbose / very verbose (shows each login:password attempt as it's tried)</td></tr><tr><td><code>-W</code></td><td>Wait time between connection attempts (seconds)</td></tr><tr><td><code>-u</code></td><td>Loop users first, then passwords (default loops passwords first for each user)</td></tr></tbody></table>

***

## Reducing Lockout Risk

Account lockout is the main operational hazard with Hydra - hammering a login form or AD account with a large wordlist can lock accounts out, generate a wall of SIEM alerts, or trip fail2ban-style protections.

```bash
hydra -l admin -P passwords.txt -t 4 -W 2 ssh://SERVER_IP
```

{% hint style="warning" %}
#### Lockout Policies

On a real engagement, always confirm the target's account lockout policy (and get client sign-off) before running any brute-force attack - this is exactly the kind of activity that should be discussed at pre-engagement, in the same way covered in the Vulnerability Scanning article's "talk to the client first" guidance.

Consider a slower, low-and-slow password spray (one or two passwords across many accounts, spaced out) instead of a full wordlist against one account where lockout policy is a concern.
{% endhint %}
