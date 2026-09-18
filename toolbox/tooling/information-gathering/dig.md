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

# dig

## Example Usage

```bash
clue@machine[/]$ dig google.com

; <<>> DiG 9.18.24-0ubuntu0.22.04.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       142.251.47.142

;; Query time: 0 msec
;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)
;; WHEN: Thu Jun 13 10:45:58 SAST 2024
;; MSG SIZE  rcvd: 54
```

## Common Commands

| Command                             | Description                                                                                                                                       |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| dig domain.com                      | Performs a default A record lookup for the domain.                                                                                                |
| dig domain.com A                    | Retrieves the IPv4 address (A record) associated with the domain.                                                                                 |
| dig domain.com AAAA                 | Retrieves the IPv6 address (AAAA record) associated with the domain.                                                                              |
| dig domain.com MX                   | Finds the mail servers (MX records) responsible for the domain.                                                                                   |
| dig domain.com NS                   | Identifies the authoritative name servers for the domain.                                                                                         |
| dig domain.com TXT                  | Retrieves any TXT records associated with the domain.                                                                                             |
| dig domain.com CNAME                | Retrieves the canonical name (CNAME) record for the domain.                                                                                       |
| dig domain.com SOA                  | Retrieves the start of authority (SOA) record for the domain.                                                                                     |
| dig @1.1.1.1 domain.com             | Specifies a specific name server to query; in this case 1.1.1.1                                                                                   |
| dig +trace domain.com               | Shows the full path of DNS resolution.                                                                                                            |
| dig -x 192.168.1.1                  | Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.                  |
| dig +short domain.com               | Provides a short, concise answer to the query.                                                                                                    |
| dig +noall +answer domain.com       | Displays only the answer section of the query output.                                                                                             |
| dig domain.com ANY                  | Retrieves all available DNS records for the domain (Note: Many DNS servers ignore ANY queries to reduce load and prevent abuse, as per RFC 8482). |
| dig axfr domain.com @ns1.domain.com | Attempts a DNS zone transfer for a domain.                                                                                                        |
