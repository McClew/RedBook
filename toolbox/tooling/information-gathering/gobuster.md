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

# Gobuster

{% hint style="info" %}
### Download & Install:

{% embed url="https://github.com/OJ/gobuster" %}

Install:

```bash
sudo apt update && sudo apt install -y gobuster
```

Or via Go directly:

```bash
go install github.com/OJ/gobuster/v3@latest
```
{% endhint %}

**Gobuster** is a fast, Go-based brute-forcing tool. It's most commonly used for directory/file discovery on web servers, but also supports DNS subdomain enumeration, virtual host discovery, and raw fuzzing modes - making it a faster, more modern alternative to older tools like [dirbuster.md](dirbuster.md "mention").

#### Practical Notes

* <mark style="color:$primary;">**No built-in recursion.**</mark> Unlike [ffuf.md](../web-application-analysis/ffuf.md "mention") or [dirbuster.md](dirbuster.md "mention"), `dir` mode doesn't recurse into discovered directories automatically - re-run against each interesting directory found manually, or script the loop.
* <mark style="color:$primary;">**Threads vs. stability.**</mark> Gobuster is fast enough that high thread counts can trip rate limiting or WAFs on real-world targets - start conservative (`-t 10`–`20`) outside of local lab environments.
* <mark style="color:$primary;">**Prefer**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**`dir`**</mark><mark style="color:$primary;">**&#x20;**</mark><mark style="color:$primary;">**for speed,**</mark> [ffuf.md](../web-application-analysis/ffuf.md "mention") <mark style="color:$primary;">**for flexibility.**</mark> Gobuster's `dir`/`vhost`/`fuzz` modes are generally faster for straightforward brute-forcing; reach for Ffuf when the fuzzing logic needs to be more complex (multiple FUZZ points, response-size filtering, recursion with depth control).

***

## Modes

Gobuster runs as subcommands rather than flags on the base binary:

<table><thead><tr><th width="177.66668701171875">Mode</th><th>Purpose</th></tr></thead><tbody><tr><td><code>dir</code></td><td>Directory/file brute-forcing over HTTP(S)</td></tr><tr><td><code>dns</code></td><td>DNS subdomain brute-forcing</td></tr><tr><td><code>vhost</code></td><td>Virtual host discovery</td></tr><tr><td><code>fuzz</code></td><td>Generic fuzzing using a <code>FUZZ</code> keyword</td></tr><tr><td><code>s3</code></td><td>Open Amazon S3 bucket enumeration</td></tr><tr><td><code>gcs</code></td><td>Open Google Cloud Storage bucket enumeration</td></tr><tr><td><code>tftp</code></td><td>TFTP filename enumeration</td></tr></tbody></table>

***

## Directory / File Brute-Forcing

{% code overflow="wrap" %}
```bash
gobuster dir -u http://SERVER_IP:PORT -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
```
{% endcode %}

### With File Extensions

{% code overflow="wrap" %}
```bash
gobuster dir -u http://SERVER_IP:PORT -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,html,txt,bak
```
{% endcode %}

### Filtering / Displaying Status Codes

```bash
# Only show these status codes
gobuster dir -u http://SERVER_IP:PORT -w wordlist.txt -s 200,204,301,302,307,401

# Exclude (blacklist) these status codes instead
gobuster dir -u http://SERVER_IP:PORT -w wordlist.txt -b 404
```

{% hint style="warning" %}
#### Page Not Found

By default, Gobuster excludes `404` responses. If a target returns a non-standard "not found" page with a `200` status (a soft-404), results will be flooded with false positives - set `-b` explicitly or use `--wildcard` handling to compensate.
{% endhint %}

### Useful Flags

<table><thead><tr><th width="144.33331298828125">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-u</code></td><td>Target URL</td></tr><tr><td><code>-w</code></td><td>Wordlist path</td></tr><tr><td><code>-x</code></td><td>File extensions to append (comma-separated, no dots)</td></tr><tr><td><code>-t</code></td><td>Threads (default: 10 — safe to push higher on local/lab targets)</td></tr><tr><td><code>-k</code></td><td>Skip TLS certificate verification</td></tr><tr><td><code>-o</code></td><td>Write output to a file</td></tr><tr><td><code>-q</code></td><td>Quiet mode — no banner</td></tr><tr><td><code>-z</code></td><td>No progress output (useful when piping/logging)</td></tr><tr><td><code>-r</code></td><td>Follow redirects</td></tr><tr><td><code>-e</code></td><td>Expanded mode — print full URLs instead of relative paths</td></tr><tr><td><code>-l</code></td><td>Show response length</td></tr><tr><td><code>-a</code></td><td>Custom User-Agent string</td></tr><tr><td><code>-c</code></td><td>Send cookies (<code>-c "PHPSESSID=..."</code>)</td></tr><tr><td><code>-H</code></td><td>Add a custom header (<code>-H "Authorization: Bearer ..."</code>)</td></tr><tr><td><code>-P</code> / <code>-U</code></td><td>Basic auth password / username</td></tr><tr><td><code>--wildcard</code></td><td>Force processing of the wordlist even if wildcard responses are detected</td></tr></tbody></table>

### With Authentication (Basic Auth)

```bash
gobuster dir -u http://SERVER_IP:PORT -w wordlist.txt -U admin -P password123
```

***

## DNS Subdomain Enumeration

{% code overflow="wrap" %}
```bash
gobuster dns -d inlanefreight.com -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
{% endcode %}

### Show IP Addresses

```bash
gobuster dns -d inlanefreight.com -w subdomains.txt -i
```

### Specify a Resolver

```bash
gobuster dns -d inlanefreight.com -w subdomains.txt -r 8.8.8.8
```

{% hint style="info" %}
#### Wildcard DNS

If a scan returns hits for effectively every word tried, the domain likely has a wildcard DNS record - Gobuster will attempt to detect and warn about this automatically, but it's worth verifying manually with a deliberately random subdomain before trusting the results.
{% endhint %}

***

## Virtual Host (VHost) Discovery

{% code overflow="wrap" %}
```bash
gobuster vhost -u http://academy.htb -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```
{% endcode %}

`--append-domain` appends the base domain to each wordlist entry automatically (so the wordlist just needs subdomain names, not full FQDNs).

### Generic Fuzzing

Fuzz mode works like Ffuf — put the `FUZZ` keyword anywhere in the URL, headers, or POST body:

```bash
gobuster fuzz -u http://SERVER_IP:PORT/FUZZ.php -w wordlist.txt
```

### S3 / GCS Bucket Enumeration

```bash
gobuster s3 -w bucket-names.txt
gobuster gcs -w bucket-names.txt
```
