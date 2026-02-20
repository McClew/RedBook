# PRTG Network Monitor Enumeration

## Discovery & Footprinting

### Nmap

We can quickly discover PRTG from an [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention") scan. It can typically be found on common web ports such as **80**, **443**, or **8080.** It is possible to change the web interface port in the Setup section when logged in as an admin.

{% code title="Scan Command" %}
```bash
sudo nmap -sV -p- --open -T4 10.129.201.50
```
{% endcode %}

{% code title="Output" %}
```shell-session
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
```
{% endcode %}

### EyeWitness

PRTG also shows up in the [eyewitness.md](../../../toolbox/tooling/web-application-analysis/eyewitness.md "mention") scan we performed earlier. Here we can see that EyeWitness lists the default credentials `prtgadmin:prtgadmin`. They are typically pre-filled on the login page, and we often find them unchanged. Vulnerability scanners such as Nessus also have [plugins](https://www.tenable.com/plugins/nessus/51874) that detect the presence of PRTG.

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Once we have discovered PRTG, we can confirm by browsing to the URL and are presented with the login page.

```
http://<IP>:8080/index.htm
```

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

***

## Enumeration

### Version Detection with cURL

We can use cURL to find PRTGs version number from the login page.

{% code title="Command" %}
```bash
curl -s http://<IP>:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```
{% endcode %}
