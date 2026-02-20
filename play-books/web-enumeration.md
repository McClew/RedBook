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

## Technology Stack Identification

{% stepper %}
{% step %}
#### Application Detection

Identify what applications - such as WordPress, Drupal, etc. - that are in use.

**Automated Analysis**

Browser tools such as [Wappalyzer](https://www.wappalyzer.com/), [urlscan.io](https://urlscan.io/) and [builtwith.com](https://builtwith.com/) can achieve this. Local automated tooling includes [whatweb.md](../toolbox/tooling/information-gathering/whatweb.md "mention") and [nikto.md](../toolbox/tooling/vulnerability-analysis/nikto.md "mention").

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

Determine what languages the server is running. If automated tools such as [Wappalyzer](https://www.wappalyzer.com/) or [whatweb.md](../toolbox/tooling/information-gathering/whatweb.md "mention") do not identify the back-end technology in use, try identifying the dependencies of discovered applications.

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

## Exploitation Techniques

### Low Hanging Fruit

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

### Broken Access Control & Authentication



### Injection Attacks

