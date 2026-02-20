---
icon: at
---

# DNS

DNS offers a variety of information about public - and sometimes private - servers, such as IP addresses, server names, and server functionality.

***

## DNS Enumeration

| Tool                       | Description                                                                                             |
| -------------------------- | ------------------------------------------------------------------------------------------------------- |
| dig                        | Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output. |
| nslookup                   | Simpler DNS lookup tool, primarily for A, AAAA, and MX records.                                         |
| host                       | Streamlined DNS lookup tool with concise output.                                                        |
| DNSEnum                    | Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).         |
| fierce                     | DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.         |
| dnsrecon                   | Combines multiple DNS reconnaissance techniques and supports various output formats.                    |
| theHarvester               | OSINT tool that gathers information from various sources, including DNS records (email addresses).      |
| Online DNS Lookup Services | User-friendly interfaces for performing DNS lookups.                                                    |

***

## Subdomain Enumeration

Subdomain enumeration is the process of systematically identifying and listing subdomains. From a DNS perspective, subdomains are typically represented by A (or AAAA for IPv6) records, which map the subdomain name to its corresponding IP address. Additionally, CNAME records might be used to create aliases for subdomains, pointing them to other domains or subdomains.

| Tool                                                                                | Description                                                                                                                                                    |
| ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [dnsenum.md](../../../toolbox/tooling/information-gathering/dnsenum.md "mention")   | Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).                                                                |
| [fierce.md](../../../toolbox/tooling/information-gathering/fierce.md "mention")     | User-friendly tool for recursive subdomain discovery, featuring wildcard detection and an easy-to-use interface.                                               |
| dnsrecon                                                                            | Versatile tool that combines multiple DNS reconnaissance techniques and offers customisable output formats.                                                    |
| [ffuf.md](../../../toolbox/tooling/web-application-analysis/ffuf.md "mention")      | Fast and flexible web fuzzer, used for discovering hidden paths, parameters, and subdomains by systematically brute-forcing URLs, headers, and other requests. |
| [gobuster.md](../../../toolbox/tooling/information-gathering/gobuster.md "mention") | Gobuster is a command-line tool for brute-forcing URIs (directories, files), DNS subdomains, and virtual hostnames on web servers.                             |
| amass                                                                               | Actively maintained tool focused on subdomain discovery, known for its integration with other tools and extensive data sources.                                |
| assetfinder                                                                         | Simple yet effective tool for finding subdomains using various techniques, ideal for quick and lightweight scans.                                              |
| puredns                                                                             | Powerful and flexible DNS brute-forcing tool, capable of resolving and filtering results effectively.                                                          |

Subdomains can also be enumerated using scripts: DNS Lookup Brute Force DNS Reverse Lookup Brute Force.

***

## Virtual Host (VHost) Enumeration

Virtual hosting is the ability of web servers to distinguish between multiple websites or applications sharing the same IP address. This is achieved by leveraging the HTTP Host header, a piece of information included in every HTTP request sent by a web browser.

| Tool                                                                                | Description                                                                                                                                                                      |
| ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [gobuster.md](../../../toolbox/tooling/information-gathering/gobuster.md "mention") | A multi-purpose tool often used for directory/file brute-forcing, but also effective for virtual host discovery. Fast, supports multiple HTTP methods, can use custom wordlists. |
| Feroxbuster                                                                         | Similar to Gobuster, but with a Rust-based implementation, known for its speed and flexibility. Supports recursion, wildcard discovery, and various filters.                     |
| [ffuf.md](../../../toolbox/tooling/web-application-analysis/ffuf.md "mention")      | Another fast web fuzzer that can be used for virtual host discovery by fuzzing the Host header. Customizable wordlist input and filtering options.                               |

***

## DNS Zone Transfer Enumeration

A DNS zone transfer is a mechanism used to replicate DNS data between DNS servers. This process includes the copying of the zone file from a master DNS server to a slave server. The zone file contains a list of all the DNS names configured for that zone. Zone transfers should be limited to authorised slave DNS servers.

A zone transfer can be requested with dig:

```bash
clue@machine[/]$ dig axfr @nsztm1.digi.ninja zonetransfer.me

; <<>> DiG 9.18.12-1~bpo11+1-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
...
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
...
;; Query time: 10 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Mon May 27 18:31:35 BST 2024
;; XFR size: 50 records (messages 1, bytes 2085)
```
