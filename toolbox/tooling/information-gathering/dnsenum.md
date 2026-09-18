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

# DNSEnum

**DNSEnum** is a versatile and widely-used command-line tool written in Perl. It is a comprehensive toolkit for DNS reconnaissance, providing various functionalities to gather information about a target domain's DNS infrastructure and potential subdomains. The tool offers several key functions:

* **DNS Record Enumeration:** dnsenum can retrieve various DNS records, including A, AAAA, NS, MX, and TXT records, providing a comprehensive overview of the target's DNS configuration.
* **Zone Transfer Attempts:** The tool automatically attempts zone transfers from discovered name servers. While most servers are configured to prevent unauthorised zone transfers, a successful attempt can reveal a treasure trove of DNS information.
* **Subdomain Brute-Forcing:** dnsenum supports brute-force enumeration of subdomains using a wordlist. This involves systematically testing potential subdomain names against the target domain to identify valid ones.
* **Google Scraping:** The tool can scrape Google search results to find additional subdomains that might not be listed in DNS records directly.
* **Reverse Lookup:** dnsenum can perform reverse DNS lookups to identify domains associated with a given IP address, potentially revealing other websites hosted on the same server.
* **WHOIS Lookups:** The tool can also perform WHOIS queries to gather information about domain ownership and registration details.
