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

# Nuclei

{% hint style="info" %}
### Download & Install:

[https://github.com/projectdiscovery/nuclei](https://github.com/projectdiscovery/nuclei)



Install

```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

Or via apt on Kali:

```bash
sudo apt update && sudo apt install -y nuclei
```
{% endhint %}

**Nuclei** is a fast, template-based vulnerability scanner from ProjectDiscovery. Rather than hardcoding checks, it runs YAML-defined templates against targets - covering CVEs, misconfigurations, exposed panels, default credentials, takeovers, and more. The template library is community-maintained and updated constantly, which is what makes Nuclei useful for keeping pace with newly disclosed vulnerabilities.

#### Practical Notes

* <mark style="color:$primary;">**Nuclei is signature-based, like any scanner.**</mark> It's excellent for known CVEs, exposures, and misconfigurations with published templates, but won't find business-logic flaws or anything without a matching template - pair it with manual testing, not instead of it.
* <mark style="color:$primary;">**Template quality varies.**</mark> The official `nuclei-templates` repo is well-maintained, but community-contributed templates can have false positives - validate hits manually before reporting them, same as with Nessus/OpenVAS findings.
* <mark style="color:$primary;">**Good for continuous/scheduled scanning.**</mark> Because it's fast and template-driven, Nuclei suits being run on a schedule against a known asset list to catch newly disclosed CVEs quickly, in a way a full Nessus scan often isn't practical for.
* <mark style="color:$primary;">**Combine with reconnaissance tools.**</mark> A common workflow is `subfinder` → `httpx` → `nuclei`, piping live subdomains straight into a scan rather than running it as a standalone step.

***

## Cheat Sheet

<table><thead><tr><th width="163">Flag</th><th>Meaning</th></tr></thead><tbody><tr><td><code>-u</code></td><td>Single target URL</td></tr><tr><td><code>-l</code></td><td>File containing a list of target URLs</td></tr><tr><td><code>-t</code></td><td>Template or template directory to run</td></tr><tr><td><code>-tags</code></td><td>Only run templates matching these tags</td></tr><tr><td><code>-exclude-tags</code></td><td>Skip templates matching these tags</td></tr><tr><td><code>-severity</code></td><td>Filter by severity (<code>info</code>, <code>low</code>, <code>medium</code>, <code>high</code>, <code>critical</code>)</td></tr><tr><td><code>-update-templates</code></td><td>Pull the latest template library</td></tr><tr><td><code>-rl</code></td><td>Rate limit — requests per second (default: 150)</td></tr><tr><td><code>-c</code></td><td>Concurrency — number of templates run in parallel</td></tr><tr><td><code>-o</code></td><td>Write output to a file</td></tr><tr><td><code>-json-export</code></td><td>Write results as JSON</td></tr><tr><td><code>-silent</code></td><td>Suppress the banner/progress noise, print results only</td></tr><tr><td><code>-H</code></td><td>Add a custom header (e.g. <code>-H "Authorization: Bearer ..."</code>)</td></tr><tr><td><code>-proxy</code></td><td>Route requests through a proxy (e.g. Burp: <code>-proxy http://127.0.0.1:8080</code>)</td></tr><tr><td><code>-debug</code></td><td>Show full request/response for troubleshooting a template</td></tr><tr><td><code>-etags</code></td><td>Exclude specific tags (older flag name — prefer <code>-exclude-tags</code> on current versions)</td></tr></tbody></table>

***

## Update Templates

Templates are stored separately from the binary and need updating in their own right:

```bash
nuclei -update-templates
```

{% hint style="warning" %}
#### Update Hygiene

Run `-update-templates` regularly. Nuclei's value comes almost entirely from an up-to-date template set - an old template library will miss recently disclosed CVEs entirely.
{% endhint %}

***

## Basic Usage

```bash
nuclei -u http://SERVER_IP:PORT
```

### Scan a List of Targets

```bash
nuclei -l targets.txt
```

### Pipe Targets In (e.g. from httpx)

```bash
cat subdomains.txt | httpx -silent | nuclei
```

***

## Selecting Templates

By default, Nuclei runs a broad set of templates. Scope this down for faster, more targeted scans.

#### By Severity

```bash
nuclei -u http://SERVER_IP:PORT -severity critical,high
```

#### By Tag

```bash
nuclei -u http://SERVER_IP:PORT -tags cve,exposure,misconfig
```

#### By Specific Template or Directory

```bash
# Single template
nuclei -u http://SERVER_IP:PORT -t http/cves/2021/CVE-2021-44228.yaml

# Whole category
nuclei -u http://SERVER_IP:PORT -t http/exposures/
```

#### Exclude Templates or Tags

```bash
nuclei -u http://SERVER_IP:PORT -exclude-tags dos,fuzz
```

***

## Rate Limiting

```bash
nuclei -u http://SERVER_IP:PORT -rl 50 -c 10
```

{% hint style="warning" %}
#### Aggresive by Default

Nuclei's default concurrency is aggressive enough to cause real load on smaller or fragile targets - as with any scanner, agree scan windows and rate limits with the client before pointing it at anything production or sensitive. Tie this back to the network impact considerations covered in the Vulnerability Scanning article.
{% endhint %}

***

### Routing Through Burp / a Proxy

```bash
nuclei -u http://SERVER_IP:PORT -proxy http://127.0.0.1:8080
```

Useful for inspecting exactly what a template sent, or for capturing traffic into a wider assessment workflow.

***

## Writing a Custom Template

Nuclei templates are plain YAML. A minimal HTTP-based example, checking for an exposed `.git` directory:

```yaml
id: exposed-git-directory

info:
  name: Exposed .git Directory
  author: yourname
  severity: medium

http:
  - method: GET
    path:
      - "{{BaseURL}}/.git/HEAD"
    matchers:
      - type: word
        words:
          - "ref:"
```

Run it directly:

```bash
nuclei -u http://SERVER_IP:PORT -t exposed-git-directory.yaml
```

{% hint style="info" %}
#### Syntax Validation

The `-validate` flag checks a template's syntax without running it - useful when iterating on a custom template: `nuclei -t mytemplate.yaml -validate`.
{% endhint %}
