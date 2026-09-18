---
icon: spider
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

# Crawling / Spidering

**Crawling**, often called **spidering**, is the automated process of systematically browsing the World Wide Web. Similar to how a spider navigates its web, a Web Crawler follows links from one page to another, collecting information. These crawlers are essentially bots that use pre-defined algorithms to discover and index web pages, making them accessible through search engines or for other purposes like data analysis and web reconnaissance.

## Web Crawlers

| Tool                            | Description                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Burp Suite Spider               | Burp Suite, a widely used web application testing platform, includes a powerful active crawler called Spider. Spider excels at mapping out web applications, identifying hidden content, and uncovering potential vulnerabilities.                                                                                                            |
| OWASP ZAP (Zed Attack Proxy)    | ZAP is a free, open-source web application security scanner. It can be used in automated and manual modes and includes a spider component to crawl web applications and identify potential vulnerabilities.                                                                                                                                   |
| Scrapy (Python Framework)       | Scrapy is a versatile and scalable Python framework for building custom web crawlers. It provides rich features for extracting structured data from websites, handling complex crawling scenarios, and automating data processing. Its flexibility makes it ideal for tailored reconnaissance tasks.                                          |
| Apache Nutch (Scalable Crawler) | Nutch is a highly extensible and scalable open-source web crawler written in Java. It's designed to handle massive crawls across the entire web or focus on specific domains. While it requires more technical expertise to set up and configure, its power and flexibility make it a valuable asset for large-scale reconnaissance projects. |

## robots.txt

**robots.txt** is a simple text file placed in the root directory of a website (e.g., _www.acme.com/robots.txt_). It adheres to the **Robots Exclusion Standard**, guidelines for how web crawlers should behave when visiting a website. This file contains instructions in the form of "directives" that tell bots which parts of the website they can and cannot crawl.

```robots.txt
User-agent: *
Disallow: /private/
```

The robots.txt file is a plain text document that lives in the root directory of a website. It follows a straightforward structure, with each set of instructions, or "record," separated by a blank line. Each record consists of two main components:

* **User-agent:** This line specifies which crawler or bot the following rules apply to. A wildcard (\*) indicates that the rules apply to all bots. Specific user agents can also be targeted, such as "Googlebot" (Google's crawler) or "Bingbot" (Microsoft's crawler).
* **Directives:** These lines provide specific instructions to the identified user-agent.

Common directives include:

| Directive   | Description                                                                                                      | Example                                                    |
| ----------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Disallow    | Specifies paths or patterns that the bot should not crawl.                                                       | Disallow: /admin/ (disallow access to the admin directory) |
| Allow       | Explicitly permits the bot to crawl specific paths or patterns, even if they fall under a broader Disallow rule. | Allow: /public/ (allow access to the public directory)     |
| Crawl-delay | Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server.              | Crawl-delay: 10 (10-second delay between requests)         |
| Sitemap     | Provides the URL to an XML sitemap for more efficient crawling.                                                  | Sitemap: https://www.acme.com/sitemap.xml                  |

For web reconnaissance, robots.txt serves as a valuable source of intelligence. While respecting the directives outlined in this file, security professionals can glean crucial insights into the structure and potential vulnerabilities of a target website:

* **Uncovering Hidden Directories:** Disallowed paths in robots.txt often point to directories or files the website owner intentionally wants to keep out of reach from search engine crawlers. These hidden areas might house sensitive information, backup files, administrative panels, or other resources that could interest an attacker.
* **Mapping Website Structure:** By analyzing the allowed and disallowed paths, security professionals can create a rudimentary map of the website's structure. This can reveal sections that are not linked from the main navigation, potentially leading to undiscovered pages or functionalities.
* **Detecting Crawler Traps:** Some websites intentionally include "honeypot" directories in robots.txt to lure malicious bots. Identifying such traps can provide insights into the target's security awareness and defensive measures.

## Well-Known URIs

The **.well-known standard**, defined in RFC 8615, serves as a standardized directory within a website's root domain. This designated location, typically accessible via the **/.well-known/** path on a web server, centralizes a website's critical metadata, including configuration files and information related to its services, protocols, and security mechanisms.

By establishing a consistent location for such data, _.well-known simplifies the discovery and access process for various stakeholders, including web browsers, applications, and security tools_. This streamlined approach enables clients to automatically locate and retrieve specific configuration files by constructing the appropriate URL. For instance, to access a website's security policy, a client would request https://acme.com/.well-known/security.txt.

The\*\* Internet Assigned Numbers Authority\*\* (**IANA**) maintains a registry of .well-known URIs, each serving a specific purpose defined by various specifications and standards. [https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml)

Below is a table highlighting a few notable examples:

| URI Suffix                   | Description                                                                                           | Status      | Reference                                                                               |
| ---------------------------- | ----------------------------------------------------------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------- |
| security.txt                 | Contains contact information for security researchers to report vulnerabilities.                      | Permanent   | RFC 9116                                                                                |
| /.well-known/change-password | Provides a standard URL for directing users to a password change page.                                | Provisional | https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri |
| openid-configuration         | Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol. | Permanent   | http://openid.net/specs/openid-connect-discovery-1\_0.html                              |
| assetlinks.json              | Used for verifying ownership of digital assets (e.g., apps) associated with a domain.                 | Permanent   | https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md     |
| mta-sts.txt                  | Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.      | Permanent   | RFC 8461                                                                                |
