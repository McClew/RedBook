---
icon: flag-checkered
---

# Initial Enumeration

## Passive Reconnaissance

Passive reconnaissance or [osint](../field-manual/intelligence/osint/ "mention"), involves gathering information without directly interacting with the target's systems, minimising detection risk.

### Domain & Host Information

{% stepper %}
{% step %}
#### WHOIS Lookup

[intelligence](../field-manual/intelligence/ "mention") > [web-enumeration](../field-manual/intelligence/web-enumeration/ "mention") > [whois.md](../field-manual/intelligence/web-enumeration/whois.md "mention")

Retrieve registration data, including creation/expiration dates, registrar, and potentially contact information (if not privacy-protected).

* [ ] Complete
{% endstep %}

{% step %}
#### DNS Record Examination

[intelligence](../field-manual/intelligence/ "mention") > [network-enumeration](../field-manual/intelligence/network-enumeration/ "mention") > [dns.md](../field-manual/intelligence/network-enumeration/dns.md "mention")

Use tools to query A, AAAA, MX, NS, and TXT records to map the infrastructure.

* [ ] Complete
{% endstep %}

{% step %}
#### Subdomain Enumeration

[intelligence](../field-manual/intelligence/ "mention") > [web-enumeration](../field-manual/intelligence/web-enumeration/ "mention") > [automated-scanning.md](../field-manual/intelligence/web-enumeration/automated-scanning.md "mention")

[intelligence](../field-manual/intelligence/ "mention") > [web-enumeration](../field-manual/intelligence/web-enumeration/ "mention") > [crawling-spidering.md](../field-manual/intelligence/web-enumeration/crawling-spidering.md "mention")

Use techniques (e.g., certificate transparency logs, brute-forcing, passive lookups) to find hidden or forgotten subdomains. Tools such as [ffuf.md](../toolbox/tooling/web-application-analysis/ffuf.md "mention") and [gobuster.md](../toolbox/tooling/information-gathering/gobuster.md "mention") can help achieve this goal.

* [ ] Complete
{% endstep %}
{% endstepper %}

### Public-Facing Assets & Code

[intelligence](../field-manual/intelligence/ "mention") > [osint](../field-manual/intelligence/osint/ "mention") > [search-engine-discovery.md](../field-manual/intelligence/osint/search-engine-discovery.md "mention")

{% stepper %}
{% step %}
#### Search Engine Dorking

Use advanced search operators (Google Dorks, Bing Dorks) to find sensitive files, error messages, login pages, or exposed directories.

* [ ] Complete
{% endstep %}

{% step %}
#### Code Repositories

Search platforms like GitHub, GitLab, and Bitbucket for exposed credentials, API keys, configuration files, or internal documentation related to the target.

* [ ] Complete
{% endstep %}

{% step %}
#### Public File Metadata

Analyse document metadata (PDFs, DOCX) found online for internal usernames, server names, or geographical data.

* [ ] Complete
{% endstep %}
{% endstepper %}

### Employee & Organisation Data

[intelligence](../field-manual/intelligence/ "mention") > [osint](../field-manual/intelligence/osint/ "mention") > [identity-hunting.md](../field-manual/intelligence/osint/identity-hunting.md "mention")

{% stepper %}
{% step %}
#### Social Media Analysis

Search for key employees on platforms like LinkedIn to understand organisational structure, technology stacks, and potential phishing targets.

* [ ] Complete
{% endstep %}

{% step %}
#### Email Harvesting

Attempt to determine a common email format for the organisation (e.g., `firstname.lastname@target.com`).

* [ ] Complete
{% endstep %}
{% endstepper %}

***

## Active Reconnaissance

Active reconnaissance involves direct interaction with the target's network, which increases the likelihood of detection but provides much richer data.

### Host Discovery & Port Scanning

{% stepper %}
{% step %}
#### Ping Sweeps/ICMP

Identify which hosts are alive on the target network.

* [ ] Complete
{% endstep %}

{% step %}
#### TCP/UDP Port Scanning (Initial)

Use a tool like [nmap](../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention") to identify common open ports and the services running on them. Start with less-invasive, non-aggressive scans.

* [ ] Complete
{% endstep %}

{% step %}
#### Firewall/IDS Evasion

Test methods to bypass basic network defences.

* [ ] Complete
{% endstep %}
{% endstepper %}

### Service & Version Enumeration

{% stepper %}
{% step %}
#### Deep Service Scanning

Re-scan the open ports identified in the initial step to accurately determine the software version, operating system, and specific configurations (e.g., HTTP headers, SSL/TLS certificate details). Ports & services can be scanned using tools like [nmap](../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention").

* [ ] Complete
{% endstep %}

{% step %}
#### Web Server Enumeration

Access identified web services to check for common files like `robots.txt`, `sitemap.xml`, or hidden directories. See [#robots.txt](../field-manual/intelligence/web-enumeration/crawling-spidering.md#robots.txt "mention") for more information.&#x20;

* [ ] Complete
{% endstep %}
{% endstepper %}

### Directory & File Enumeration

{% stepper %}
{% step %}
#### Content Discovery (Fuzzing/Brute-forcing)

Use tools like [ffuf.md](../toolbox/tooling/web-application-analysis/ffuf.md "mention") or [gobuster.md](../toolbox/tooling/information-gathering/gobuster.md "mention") against identified web services to find hidden files, folders, or pages that are not linked from the main site.

* [ ] Complete
{% endstep %}

{% step %}
#### Virtual Host Enumeration

Check for multiple websites hosted on the same IP address.

* [ ] Complete
{% endstep %}
{% endstepper %}
