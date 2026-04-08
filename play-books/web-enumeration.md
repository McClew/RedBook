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
---

# Web Enumeration

## If / Then Guide

| If Enumeration Reveals...                                                     | ...Focus on these Attacks...                                                                                                                                                                                                                                                                 | ...and perform these Actions                                                                                                                                                               |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p>URL Parameters</p><p>(e.g., <code>?id=1</code>, <code>?cat=web</code>)</p> | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/sql-injection-sqli/">sql-injection-sqli</a></li><li><a data-mention href="../field-manual/exploitation/web-attacks/insecure-direct-object-references-idor.md">insecure-direct-object-references-idor.md</a></li></ul> | <ul><li>Test <code>id=1'</code> for SQL errors.</li><li>Change <code>user_id=10</code> to <code>user_id=11</code> to check for other users' data.</li></ul>                                |
| <p>File Paths in URLs</p><p>(e.g., <code>?page=contact.php</code>)</p>        | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/path-traversal.md">path-traversal.md</a></li><li><a data-mention href="../field-manual/exploitation/web-attacks/file-inclusion.md">file-inclusion.md</a></li></ul>                                                    | <ul><li>Try path traversal payloads (e.g., <code>../../etc/passwd</code>).</li><li>Check if you can include remote scripts via RFI.</li></ul>                                              |
| Login Panels & Forms                                                          | <ul><li>Weak/Default Credentials</li><li><a data-mention href="../field-manual/exploitation/web-attacks/sql-injection-sqli/">sql-injection-sqli</a></li></ul>                                                                                                                                | <ul><li>Try <code>admin:admin</code> or <code>guest:guest</code>.</li><li>Research the "Default Configuration" for that specific software.</li></ul>                                       |
| <p>Input Fields</p><p>(Search, Comments, Profile)</p>                         | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/cross-site-scripting-xss/">cross-site-scripting-xss</a></li><li><a data-mention href="../field-manual/exploitation/web-attacks/command-injection.md">command-injection.md</a></li></ul>                               | <ul><li>Inject <code>&#x3C;script>alert(1)&#x3C;/script></code> to check for reflection.</li><li>Try system command delimiters like <code>; whoami</code> or <code>|| id</code>.</li></ul> |
| File Upload Buttons & Inputs                                                  | <ul><li>Misconfigured <a data-mention href="../field-manual/exploitation/web-attacks/file-upload.md">file-upload.md</a></li></ul>                                                                                                                                                            | <ul><li>Attempt to upload a <code>.php</code> or <code>.phtml</code> web shell.</li><li>Try bypassing extension filters (e.g., <code>.jpg.php</code> or uppercase extensions).</li></ul>   |
| <p>Specific Frameworks</p><p>(Jinja2, Twig, Smarty)</p>                       | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/server-side-template-injection-ssti.md">server-side-template-injection-ssti.md</a></li></ul>                                                                                                                          | <ul><li>Use payloads like <code>{{7*7}}</code> to see if the server returns <code>49</code>.</li><li>Search for RCE payloads specific to that engine.</li></ul>                            |
| XML or API Endpoints                                                          | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/xml-external-entity-xxe-injection/">xml-external-entity-xxe-injection</a></li></ul>                                                                                                                                   | <ul><li>Inject a <code>DOCTYPE</code> to try and read local system files.</li><li>Check if the parser interacts with internal services.</li></ul>                                          |
| URL Inputs & Webhooks                                                         | <ul><li><a data-mention href="../field-manual/exploitation/web-attacks/server-side-request-forgery-ssrf.md">server-side-request-forgery-ssrf.md</a></li></ul>                                                                                                                                | <ul><li>Force the server to request <code>http://localhost:80</code> or internal metadata IPs.</li></ul>                                                                                   |

***

## Technology Stack Identification

{% stepper %}
{% step %}
#### Application Detection

Identify what applications - such as WordPress, Drupal, etc. - that are in use.

**Automated Analysis**

Browser tools such as [Wappalyzer](https://www.wappalyzer.com/), [urlscan.io](https://urlscan.io/) and [builtwith.com](https://builtwith.com/) can achieve this. Local automated tooling includes [whatweb.md](../toolbox/tooling/web-application-analysis/whatweb.md "mention") and [nikto.md](../toolbox/tooling/vulnerability-analysis/nikto.md "mention").

**Manual Analysis**

Manual analysis can be conducted using `curl` and viewing headers or grepping for keywords or simply review the source code via the browser console.

* [ ] Detect applications

***
{% endstep %}

{% step %}
#### Application Version Identification

[intelligence](../field-manual/intelligence/ "mention") > [application-enumeration](../field-manual/intelligence/application-enumeration/ "mention")

The tools and techniques mentioned above will likely also provide the version of the application in use, however, if deeper investigation is required utilise the applications entry within [application-enumeration](../field-manual/intelligence/application-enumeration/ "mention") if it exists.

* [ ] Identify application versions

***
{% endstep %}

{% step %}
#### Technology Discovery

Determine what languages the server is running. If automated tools such as [Wappalyzer](https://www.wappalyzer.com/) or [whatweb.md](../toolbox/tooling/web-application-analysis/whatweb.md "mention") do not identify the back-end technology in use, try identifying the dependencies of discovered applications.

* [ ] Discover the whole technology stack

***
{% endstep %}

{% step %}
#### Default Configuration Research

Research the default configurations and credentials for key technology/applications discovered.

* [ ] Find and record default configuration information

***
{% endstep %}
{% endstepper %}

***

## Path Enumeration

{% stepper %}
{% step %}
#### Common Files

Check `robots.txt` and `sitemap.xml` for hidden directories or restricted paths.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Fuzzing

Use tools like [ffuf.md](../toolbox/tooling/web-application-analysis/ffuf.md "mention") and [gobuster.md](../toolbox/tooling/information-gathering/gobuster.md "mention") (see [Broken link](/broken/pages/ro71vMH77bA4elk2JCF3 "mention") > [information-gathering](../toolbox/tooling/information-gathering/ "mention") for more) to find hidden files, folders and pages not directly linked on the site.

With the technology of the site analysed, we can fuzz using specific file extensions such as `.php`.

* [ ] Files fuzzed
* [ ] Directories fuzzed

***
{% endstep %}
{% endstepper %}

***

## Automated Scanning

{% stepper %}
{% step %}
#### Automated Mapping

[Broken link](/broken/pages/ro71vMH77bA4elk2JCF3 "mention") > [tooling](../toolbox/tooling/ "mention") > [web-application-analysis](../toolbox/tooling/web-application-analysis/ "mention")

Run tools like [eyewitness.md](../toolbox/tooling/web-application-analysis/eyewitness.md "mention") to generate a visual gallery of web interfaces for easy identification of high-value targets.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Automated Auditing

[Broken link](/broken/pages/ro71vMH77bA4elk2JCF3 "mention") > [tooling](../toolbox/tooling/ "mention") > [vulnerability-analysis](../toolbox/tooling/vulnerability-analysis/ "mention")

Utilise tools such as [nikto.md](../toolbox/tooling/vulnerability-analysis/nikto.md "mention") against specific targets to identify low-hanging fruit such as common misconfigurations.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Automated Defence Discovery

[Broken link](/broken/pages/ro71vMH77bA4elk2JCF3 "mention") > [tooling](../toolbox/tooling/ "mention") > [web-application-analysis](../toolbox/tooling/web-application-analysis/ "mention")

Use automated tools such as [wafw00f.md](../toolbox/tooling/web-application-analysis/wafw00f.md "mention") to identify and fingerprint Web Application Firewalls (WAF) that are active on the target.

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Exploitation Techniques

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention")

### Low-Hanging Fruit

{% stepper %}
{% step %}
#### Weak Credentials

Attempt common weak credentials, such as `admin`:`admin` on found login panels.

_Probability of success increases if application/web server is in 'development' and should not be public._

* [ ] Complete

***
{% endstep %}

{% step %}
#### Default Credentials

Using information found from researching the targets technology stack, attempt known default credentials for the application.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Known Exploits

Using tools such as [Exploit-DB](https://www.exploit-db.com/), or [searchsploit.md](../toolbox/tooling/exploitation-tools/searchsploit.md "mention") to search for public exploits for identified technology.

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### Access Control & Authentication

{% stepper %}
{% step %}
#### Broken Access Controls

Test for direct access to administrative URLs or unauthorised data.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Broken Authentication

Audit for weak session management & guessable tokens.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Insecure Direct Object Reference (IDOR)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [insecure-direct-object-references-idor.md](../field-manual/exploitation/web-attacks/insecure-direct-object-references-idor.md "mention")

Audit for incorrectly implemented access control checks by modifying paramaters (e.g. `user_id`) to access data without authorisation.

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### Injection Vulnerabilities

{% stepper %}
{% step %}
#### SQL Injection (SQLi)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [sql-injection-sqli](../field-manual/exploitation/web-attacks/sql-injection-sqli/ "mention")

Test input fields and URL parameters for database query manipulation.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Command Injection

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [command-injection.md](../field-manual/exploitation/web-attacks/command-injection.md "mention")

Prove for the ability to execuite system commands via the web application.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Cross-Site Scripting (XSS)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [Broken link](/broken/pages/hal1J22c3bmQ0FG4obsI "mention")

Audit for user inputs that are reflected in the browser without sanitisation.

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### File & Resource Manipulation

{% stepper %}
{% step %}
#### Directory Traversal

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [path-traversal.md](../field-manual/exploitation/web-attacks/path-traversal.md "mention")

Attempt to access sensitive system files such as `/etc/passwd`.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Local & Remote File Inclusion (LFI / RFI)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [file-inclusion.md](../field-manual/exploitation/web-attacks/file-inclusion.md "mention")

Prove for the ability to include local or remote files into the application execution.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Misconfigured File Upload

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [file-upload.md](../field-manual/exploitation/web-attacks/file-upload.md "mention")

Attempt to bypass extension filters to upload web shells for [remote-code-execution.md](../field-manual/exploitation/remote-code-execution.md "mention").

* [ ] Complete

***
{% endstep %}
{% endstepper %}

### Server-Side Vulnerabilities

{% stepper %}
{% step %}
#### Server-Side Request Forgery (SSRF)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [server-side-request-forgery-ssrf.md](../field-manual/exploitation/web-attacks/server-side-request-forgery-ssrf.md "mention")

Audit the application for parameters that can be manipulated to force the server into making unauthorised requests to internal resources or cloud services.

* [ ] Complete

***
{% endstep %}

{% step %}
#### XML External Entity (XXE) Injection

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [xml-external-entity-xxe-injection](../field-manual/exploitation/web-attacks/xml-external-entity-xxe-injection/ "mention")

Test XML input fields and parsers for the ability to reference external entities to read local system files or interact with internal services.

* [ ] Complete

***
{% endstep %}

{% step %}
#### Server-Side Template Injection (SSTI)

[exploitation](../field-manual/exploitation/ "mention") > [web-attacks](../field-manual/exploitation/web-attacks/ "mention") > [server-side-template-injection-ssti.md](../field-manual/exploitation/web-attacks/server-side-template-injection-ssti.md "mention")

Identify and probe web template engines (e.g. `Jinga2` and `Twig`) for the ability to inject and execute malicious code within the servers context.

* [ ] Complete

***
{% endstep %}
{% endstepper %}
