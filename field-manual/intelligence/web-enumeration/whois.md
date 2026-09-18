---
icon: question
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

# WHOIS

WHOIS is a widely used query and response protocol designed to access databases that store information about registered internet resources. Primarily associated with domain names, WHOIS can also provide details about IP address blocks and autonomous systems.

```bash
clue@machine[/]$ whois acme.com

[...]
Domain Name: acme.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
[...]
```

Each WHOIS record typically contains the following information:

* **Domain Name:** The domain name itself (e.g., example.com)
* **Registrar:** The company where the domain was registered (e.g., GoDaddy, Namecheap)
* **Registrant Contact:** The person or organization that registered the domain.
* **Administrative Contact:** The person responsible for managing the domain.
* **Technical Contact:** The person handling technical issues related to the domain.
* **Creation & Expiration Dates:** When the domain was registered and when it's set to expire.
* **Name Servers:** Servers that translate the domain name into an IP address.
