---
icon: memo-circle-check
---

# Note Taking

### Administrative & Environmental Markers

{% stepper %}
{% step %}
#### Foothold Established

Document the exact timestamp, the user account compromised, the source IP, and the method of entry.
{% endstep %}

{% step %}
#### Credentials/Hashes Discovered

Create a dedicated "Credential Table" immediately upon finding any password, hash, or API key. Include the source (e.g., config file, memory dump) and where it was found.
{% endstep %}

{% step %}
#### Network Pivot

If you gain access to a new subnet or interface, record the new IP range and the "dual-homed" host acting as the bridge.
{% endstep %}
{% endstepper %}

### Enumeration & Discovery Markers

{% stepper %}
{% step %}
#### Unusual Port/Service

When an Nmap scan reveals a non-standard port (e.g., 8888, 10000), note the service banner and any manual "prodding" results.
{% endstep %}

{% step %}
#### Application Versioning

Record every software version found (e.g., Apache 2.4.41). This is vital for the "Vulnerability Analysis" section of your report.
{% endstep %}

{% step %}
#### Hidden Content/Vhost

When `ffuf` or `gobuster` finds a 200 OK on a hidden directory or virtual host, log the discovery and the wordlist used.
{% endstep %}
{% endstepper %}

### Exploitation & Evidence Markers

{% stepper %}
{% step %}
#### Successful Exploit

Take a screenshot of the command used and the successful output. Note any modifications made to public exploit code (e.g., changing a payload or offset).
{% endstep %}

{% step %}
#### "Rabbit Hole" Abandonment

If you spend more than 30 minutes on a path that fails, document _why_ you think it failed. This prevents you from looping back to the same dead-end later.
{% endstep %}

{% step %}
#### Security Bypass

Document any time you have to bypass a firewall, IDS, or WAF. Include the specific technique used (e.g., header manipulation, encoding).
{% endstep %}
{% endstepper %}

### Privilege Escalation Markers

{% stepper %}
{% step %}
#### Misconfiguration Identified

Log the specific finding (e.g., SUID binary, writable `/etc/passwd`, or `sudo -l` entry).
{% endstep %}

{% step %}
#### GTFOBins/Payload Reference

Note the specific URL or payload used from external sources like GTFOBins or PayloadsAllTheThings.
{% endstep %}

{% step %}
#### Post-Exploitation Cleanup

Keep a running list of every file you upload (e.g., `linpeas.sh`, `nc`) and every user you add. This ensures you can revert the system to its original state.
{% endstep %}
{% endstepper %}

### Reporting & Impact Markers

{% stepper %}
{% step %}
#### Sensitive Data Access

Note any PII (Personally Identifiable Information), configuration files, or internal documentation you can read.
{% endstep %}

{% step %}
#### Impact Demonstration

Document what an attacker could do with your level of access (e.g., "Access to the database allows for the full exfiltration of the customer list").
{% endstep %}
{% endstepper %}
