---
icon: fingerprint
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

# Fingerprinting

Fingerprinting focuses on extracting technical details about the technologies powering a website or web application. The digital signatures of web servers, operating systems, and software components can reveal critical information about a target's infrastructure and potential security weaknesses. This knowledge empowers attackers to tailor attacks and exploit vulnerabilities specific to the identified technologies.

Fingerprinting serves as a cornerstone of web reconnaissance for several reasons:

* **Targeted Attacks:** By knowing the specific technologies in use, attackers can focus their efforts on exploits and vulnerabilities that are known to affect those systems. This significantly increases the chances of a successful compromise.
* **Identifying Misconfigurations:** Fingerprinting can expose misconfigured or outdated software, default settings, or other weaknesses that might not be apparent through other reconnaissance methods.
* **Prioritising Targets:** When faced with multiple potential targets, fingerprinting helps prioritise efforts by identifying systems more likely to be vulnerable or hold valuable information.
* **Building a Comprehensive Profile:** Combining fingerprint data with other reconnaissance findings creates a holistic view of the target's infrastructure, aiding in understanding its overall security posture and potential attack vectors.

## Fingerprinting Techniques

There are several techniques used for web server and technology fingerprinting:

* **Banner Grabbing:** Banner grabbing involves analysing the banners presented by web servers and other services. These banners often reveal the server software, version numbers, and other details.
* **Analysing HTTP Headers:** HTTP headers transmitted with every web page request and response contain a wealth of information. The Server header typically discloses the web server software, while the X-Powered-By header might reveal additional technologies like scripting languages or frameworks.
* **Probing for Specific Responses:** Sending specially crafted requests to the target can elicit unique responses that reveal specific technologies or versions. For example, certain error messages or behaviours are characteristic of particular web servers or software components.
* **Analysing Page Content:** A web page's content, including its structure, scripts, and other elements, can often provide clues about the underlying technologies. There may be a copyright header that indicates specific software being used, for example.

## Fingerprinting Tools

| Tool       | Description                                                                                                           | Features                                                                                            |
| ---------- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Wappalyzer | Browser extension and online service for website technology profiling.                                                | Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more. |
| BuiltWith  | Web technology profiler that provides detailed reports on a website's technology stack.                               | Offers both free and paid plans with varying levels of detail.                                      |
| WhatWeb    | Command-line tool for website fingerprinting.                                                                         | Uses a vast database of signatures to identify various web technologies.                            |
| Nmap       | Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting. | Can be used with scripts (NSE) to perform more specialised fingerprinting.                          |
| Netcraft   | Offers a range of web security services, including website fingerprinting and security reporting.                     | Provides detailed reports on a website's technology, hosting provider, and security posture.        |
| wafw00f    | Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).                             | Helps determine if a WAF is present and, if so, its type and configuration.                         |

## Banner Grabbing

Use the curl command with the -I flag (or --head) to fetch only the HTTP headers, not the entire page content.

```bash
clue@machine[/]$ curl -I acme.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 19 June 2026 12:07:44 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://acme.com/
Content-Type: text/html; charset=iso-8859-1
```

This command shows that the acme.com is running on Apache 2.4.41 and is attempting to redirect to https://acme.com.

```bash
clue@machine[/]$ curl -I https://acme.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:12:12 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Redirect-By: WordPress
Location: https://www.acme.com/
Content-Type: text/html; charset=UTF-8
```

Now we've found that WordPress is being used and attempting another redirect to https://www.acme.com.

```bash
clue@machine[/]$ curl -I https://www.acme.com

HTTP/1.1 200 OK
Date: Fri, 31 May 2024 12:12:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.acme.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.acme.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.acme.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8
```
