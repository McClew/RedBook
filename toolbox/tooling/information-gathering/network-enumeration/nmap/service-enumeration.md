# Service Enumeration

## Service Version Detection

```bash
clue@machine[~]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:44 CEST
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

| **Scanning Options** | **Description**                                        |
| -------------------- | ------------------------------------------------------ |
| `-sV`                | Performs service version detection on specified ports. |

## Web Discovery

```bash
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list
```
