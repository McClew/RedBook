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

# Wafw00f

| Download                                  |
| ----------------------------------------- |
| https://github.com/EnableSecurity/wafw00f |

WAFW00F allows one to identify and fingerprint Web Application Firewall (WAF) products protecting a website.

## Install

```bash
pip3 install git+https://github.com/EnableSecurity/wafw00f
```

## Basic Usage

```bash
clue@machine[/]$ wafw00f acme.com

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://acme.com
[+] The site https://acme.com is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2

```
