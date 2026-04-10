---
icon: ethernet
---

# 21 - FTP

The **File Transfer Protocol (FTP)** serves as a standard protocol for file transfer across a computer network between a server and a client.

{% hint style="info" %}
The default ports for FTP are TCP port 21 for control commands and TCP port 20 for data transfer.
{% endhint %}

It is a **plain-text** protocol that uses as **new line character `0x0d 0x0a`** so sometimes you need to **connect using `telnet`** or **`nc -C`**.

```bash
PORT   STATE SERVICE
21/tcp open  ftp
```

Common anonymous login credentials include:

* `anonymous`:`anonymous`
* `anonymous`:\<blank>
* `ftp`:`ftp`
* `guest`:`guest`

***

## Cheatsheet

{% tabs %}
{% tab title="Tools" %}
| Action                                                                                                                | Description                                                                                                          |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `sudo nmap -sC -sV -p 21 -v <target>`                                                                                 | Performs an Nmap scan on the FTP service to identify versions, scripts, and checks for anonymous login. Quite noisy. |
| <p><code>sudo nmap -sV -p21 --script</code>   </p><p><code>ftp-anon &#x3C;target-ip></code></p>                       | Runs an Nmap script to check for anonymous authentication on the target FTP server.                                  |
| <p><code>ftp &#x3C;target></code><br><code>nc -nv &#x3C;target> 21</code><br><code>telnet &#x3C;target> 21</code></p> | Different ways to connect to a remote FTP service.                                                                   |
| `wget -m --no-passive ftp://<user>:<password>@<target-ip>`                                                            | Recursively downloads all accessible files from the target FTP server using Wget.                                    |
{% endtab %}

{% tab title="Commands" %}
<table><thead><tr><th width="167.4000244140625">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>?</code></td><td>Request help or information about the FTP commands.</td></tr><tr><td><code>bye</code><br><code>quit</code></td><td>Exit the FTP environment.</td></tr><tr><td><code>cd</code></td><td>Change directory on the remote machine.</td></tr><tr><td><code>delete</code></td><td>Delete a file in the current remote directory.</td></tr><tr><td><code>get &#x3C;FILE></code></td><td>Copy one file from the remote machine to the local machine.</td></tr><tr><td><code>get &#x3C;FILE> /local/path</code></td><td>Copies file in the current remote directory to a file named in our current local directory.</td></tr><tr><td><code>lcd</code></td><td>Change directory on your local machine.</td></tr><tr><td><code>ls</code></td><td>List the names of the files in the current remote directory.</td></tr><tr><td><code>mkdir</code></td><td>Make a new directory within the current remote directory.</td></tr><tr><td><code>mget</code></td><td>Copy multiple files from the remote machine to the local machine;<br>prompted for a y/n answer before transferring each file.</td></tr><tr><td><code>mput</code></td><td>Copy multiple files from the local machine to the remote machine;<br>prompted for a y/n answer before transferring each file.</td></tr><tr><td><code>put</code></td><td>Copy one file from the local machine to the remote machine.</td></tr><tr><td><code>rmdir</code></td><td>Delete a directory in the current remote directory.</td></tr></tbody></table>
{% endtab %}
{% endtabs %}

***

## Banner Grabbing

```bash
nc -vn <IP> 21
openssl s_client -connect crossfit.htb:21 -starttls ftp # Get certificate if any
```

***

## Connect to FTP using starttls <a href="#connect-to-ftp-using-starttls" id="connect-to-ftp-using-starttls"></a>

```bash
lftp
lftp :~> set ftp:ssl-force true
lftp :~> set ssl:verify-certificate no
lftp :~> connect 10.10.10.208
lftp 10.10.10.208:~> login
Usage: login <user|URL> [<pass>]
lftp 10.10.10.208:~> login username Password
```

***

## Unauth enum <a href="#unauth-enum" id="unauth-enum"></a>

With **nmap**

```bash
sudo nmap -sV -p21 -sC -A 10.10.10.10
```

You can us the commands `HELP` and `FEAT` to obtain some information of the FTP server:

```
HELP
214-The following commands are recognized (* =>'s unimplemented):
214-CWD     XCWD    CDUP    XCUP    SMNT*   QUIT    PORT    PASV
214-EPRT    EPSV    ALLO*   RNFR    RNTO    DELE    MDTM    RMD
214-XRMD    MKD     XMKD    PWD     XPWD    SIZE    SYST    HELP
214-NOOP    FEAT    OPTS    AUTH    CCC*    CONF*   ENC*    MIC*
214-PBSZ    PROT    TYPE    STRU    MODE    RETR    STOR    STOU
214-APPE    REST    ABOR    USER    PASS    ACCT*   REIN*   LIST
214-NLST    STAT    SITE    MLSD    MLST
214 Direct comments to root@drei.work

FEAT
211-Features:
 PROT
 CCC
 PBSZ
 AUTH TLS
 MFF modify;UNIX.group;UNIX.mode;
 REST STREAM
 MLST modify*;perm*;size*;type*;unique*;UNIX.group*;UNIX.mode*;UNIX.owner*;
 UTF8
 EPRT
 EPSV
 LANG en-US
 MDTM
 SSCN
 TVFS
 MFMT
 SIZE
211 End

STAT
#Info about the FTP server (version, configs, status...)
```

***

## Anonymous login <a href="#anonymous-login" id="anonymous-login"></a>

_anonymous : anonymous_\
\_anonymous :\
&#xNAN;_\_ftp : ftp_

```bash
ftp <IP>
>anonymous
>anonymous
>ls -a   # List all files (even hidden) (yes, they could be hidden)
>binary  # Set transmission to binary instead of ascii
>ascii   # Set transmission to ascii instead of binary
>bye     # Exit
```

***

## Brute force

Here you can find a nice list with default ftp credentials:

[https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt)

***

## Automated <a href="#automated" id="automated"></a>

Anon login and bounce FTP checks are perform by default by `nmap` with **`-sC`** option or:

```bash
nmap --script ftp-* -p 21 <ip>
```
