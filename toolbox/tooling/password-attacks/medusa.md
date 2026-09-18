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

# Medusa

{% hint style="info" %}
#### Download & Install:

{% embed url="https://github.com/jmk-foofus/medusa" %}

Install:

```bash
sudo apt update && sudo apt install -y medusa
```
{% endhint %}

**Medusa** is a fast, modular, parallelised login brute-forcer - functionally similar to [hydra.md](hydra.md "mention"), and the two are often interchangeable in practice. Medusa's module-based architecture (each protocol is a separate `.mod` file) makes it straightforward to extend.

#### Practical Notes

* <mark style="color:$primary;">**Check module-specific options with**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**`-d`**</mark><mark style="color:$primary;">**.**</mark> Many modules (SMB variants especially) take extra module parameters via `-m` that aren't obvious from the main help output - `medusa -M <module> -d` (or checking the module's own help) is worth doing before assuming default behaviour is correct.
* <mark style="color:$primary;">**`smbnt`**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**vs. other SMB modules.**</mark> Medusa has historically had multiple SMB-related modules; `smbnt` is the one generally used for standard NTLM authentication testing - confirm against the installed version's `-d` output, since module names can shift between releases.
* <mark style="color:$primary;">**Verify hits manually**</mark>, exactly as with Hydra - automated brute-forcers can produce false positives against inconsistent web error pages.

***

## Cheat Sheet

### List Available Modules

```bash
medusa -d
```

### Basic Syntax

```bash
medusa -h <target> -u <username> -P <wordlist> -M <module>
```

### Specifying Credentials

| Flag    | Meaning                                                                 |
| ------- | ----------------------------------------------------------------------- |
| `-u`    | Single username                                                         |
| `-U`    | File containing a list of usernames                                     |
| `-p`    | Single password                                                         |
| `-P`    | File containing a list of passwords                                     |
| `-C`    | Combo file, `username:password` per line                                |
| `-e ns` | Additional password checks: `n` = null password, `s` = same as username |

{% hint style="info" %}
#### Additional Password Checks

Medusa's `-e` only supports `n` and `s` - unlike Hydra's `-e nsr`, there's no built-in reversed-username check. Add it as an explicit entry in the password list instead if needed.
{% endhint %}

### Specifying Targets

<table><thead><tr><th width="127">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-h</code></td><td>Single target host</td></tr><tr><td><code>-H</code></td><td>File containing a list of target hosts</td></tr><tr><td><code>-M</code></td><td>Module to use (protocol)</td></tr><tr><td><code>-m</code></td><td>Pass a module-specific parameter (repeatable)</td></tr><tr><td><code>-n</code></td><td>Non-default port</td></tr></tbody></table>

***

## Protocol Examples

#### SSH

{% code overflow="wrap" %}
```bash
medusa -h SERVER_IP -u root -P /opt/useful/seclists/Passwords/darkweb2017-top10000.txt -M ssh
```
{% endcode %}

#### FTP

```bash
medusa -h SERVER_IP -U users.txt -P passwords.txt -M ftp
```

#### RDP

```bash
medusa -h SERVER_IP -u administrator -P passwords.txt -M rdp
```

#### SMB (via smbnt module)

```bash
medusa -h SERVER_IP -U users.txt -P passwords.txt -M smbnt
```

#### MySQL

```bash
medusa -h SERVER_IP -u root -P passwords.txt -M mysql
```

#### Telnet

```bash
medusa -h SERVER_IP -u admin -P passwords.txt -M telnet
```

#### POP3 / IMAP

```bash
medusa -h SERVER_IP -u user@inlanefreight.com -P passwords.txt -M pop3
medusa -h SERVER_IP -u user@inlanefreight.com -P passwords.txt -M imap
```

#### VNC

```bash
medusa -h SERVER_IP -P passwords.txt -M vnc
```

#### HTTP (via the web-form module)

Medusa's web-form module takes its parameters through repeated `-m` flags rather than a single delimited string like Hydra's `http-post-form`:

{% code overflow="wrap" %}
```bash
medusa -h SERVER_IP -u admin -P passwords.txt -M web-form -m FORM:"/login.php" -m FORM-DATA:"username=&password=" -m DENY-SIGNAL:"Invalid credentials"
```
{% endcode %}

{% hint style="info" %}
#### String Match

As with Hydra's `F=`/`S=`, Medusa's web-form module supports matching on either a failure string (`DENY-SIGNAL`) or a success string (`SUCCESS-SIGNAL`) - prefer whichever gives a cleaner, more unique match against the target's actual responses.
{% endhint %}

***

## Performance & Behaviour Flags

<table><thead><tr><th width="104.33331298828125">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-t</code></td><td>Total number of parallel logins across all hosts</td></tr><tr><td><code>-T</code></td><td>Total number of parallel hosts to test at once</td></tr><tr><td><code>-f</code></td><td>Stop scanning this host after first valid username/password pair found</td></tr><tr><td><code>-F</code></td><td>Stop entirely after first valid username/password pair found on any host</td></tr><tr><td><code>-n</code></td><td>Non-default port</td></tr><tr><td><code>-O</code></td><td>Write output to a file</td></tr><tr><td><code>-v</code></td><td>Verbosity level (0–6)</td></tr><tr><td><code>-w</code></td><td>Wildcard/timing option for some modules — check <code>-d</code> output per-module for specifics</td></tr></tbody></table>

***

## Reducing Lockout Risk

Medusa's parallelism is exactly what makes it effective and exactly what risks tripping account lockout policies or alerting a SOC.

```bash
medusa -h SERVER_IP -u admin -P passwords.txt -M ssh -t 4
```

{% hint style="warning" %}
#### Confirm Lockout Policies

Confirm lockout policy and get sign-off before brute-forcing on a real engagement - the same pre-engagement guidance in the Hydra and Vulnerability Scanning articles applies here without modification. A low-and-slow password spray across many accounts is often the safer choice over a full wordlist against one account.
{% endhint %}

***

## Medusa vs. Hydra

|                                  | Medusa                                                                | Hydra                                    |
| -------------------------------- | --------------------------------------------------------------------- | ---------------------------------------- |
| Module discovery                 | `medusa -d`                                                           | `hydra -U` (lists service help)          |
| HTTP form syntax                 | Repeated `-m` parameters                                              | Single delimited module string           |
| Built-in reversed-username check | No                                                                    | Yes (`-e r`)                             |
| Community/module ecosystem       | Smaller, but easy to extend (each module is a standalone `.mod` file) | Larger, more actively maintained overall |

In practice, most testers default to Hydra and reach for Medusa mainly when a specific module behaves better against a particular target, or out of habit/preference - there's rarely a hard requirement to use one over the other.
