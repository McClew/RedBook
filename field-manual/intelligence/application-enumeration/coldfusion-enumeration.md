# ColdFusion Enumeration

## Discovery & Footprinting

During a penetration testing enumeration, several ways exist to identify whether a web application uses ColdFusion. Here are some methods that can be used:

<table data-header-hidden><thead><tr><th width="146">Method</th><th>Description</th></tr></thead><tbody><tr><td><code>Port Scanning</code></td><td>ColdFusion typically uses port 80 for HTTP and port 443 for HTTPS by default. So, scanning for these ports may indicate the presence of a ColdFusion server. Nmap might be able to identify ColdFusion during a services scan specifically.</td></tr><tr><td><code>File Extensions</code></td><td>ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.</td></tr><tr><td><code>HTTP Headers</code></td><td>Check the HTTP response headers of the web application. ColdFusion typically sets specific headers, such as "Server: ColdFusion" or "X-Powered-By: ColdFusion", that can help identify the technology being used.</td></tr><tr><td><code>Error Messages</code></td><td>If the application uses ColdFusion and there are errors, the error messages may contain references to ColdFusion-specific tags or functions.</td></tr><tr><td><code>Default Files</code></td><td>ColdFusion creates several default files during installation, such as "admin.cfm" or "CFIDE/administrator/index.cfm". Finding these files on the web server may indicate that the web application runs on ColdFusion.</td></tr></tbody></table>

### Port Scanning

ColdFusion typically uses port 80 for HTTP and port 443 for HTTPS by default. So, scanning for these ports may indicate the presence of a ColdFusion server. Nmap might be able to identify ColdFusion during a services scan specifically.

ColdFusion exposes a fair few ports by default:

<table><thead><tr><th width="136">Port Number</th><th width="117">Protocol</th><th>Description</th></tr></thead><tbody><tr><td>80</td><td>HTTP</td><td>Used for non-secure HTTP communication between the web server and web browser.</td></tr><tr><td>443</td><td>HTTPS</td><td>Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.</td></tr><tr><td>1935</td><td>RPC</td><td>Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.</td></tr><tr><td>25</td><td>SMTP</td><td>Simple Mail Transfer Protocol (SMTP) is used for sending email messages.</td></tr><tr><td>8500</td><td>SSL</td><td>Used for server communication via Secure Socket Layer (SSL).</td></tr><tr><td>5500</td><td>Server Monitor</td><td>Used for remote administration of the ColdFusion server.</td></tr></tbody></table>

Example scan:

{% code title="Command" %}
```bash
nmap -p- -sC -Pn 10.129.247.30 --open
```
{% endcode %}

{% code title="Output" %}
```shell-session
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-13 11:45 GMT
Nmap scan report for 10.129.247.30
Host is up (0.028s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 350.38 seconds
```
{% endcode %}

The port scan results show a service running on `8500` the default port that ColdFusion uses for SSL. Navigating to the `IP:8500` lists 2 directories, `CFIDE` and `cfdocs,` in the root, further indicating that ColdFusion is running on port 8500.

### File Extensions

ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

The `/CFIDE/administrator` path loads the ColdFusion 8 Administrator login page. Now we know for certain that `ColdFusion 8` is running on the server.
