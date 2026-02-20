# Ffuf

{% hint style="info" %}
## Download & Install:

[https://github.com/ffuf/ffuf](https://github.com/ffuf/ffuf)
{% endhint %}

## Directory Fuzzing

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```
{% endcode %}

## Page Fuzzing

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```
{% endcode %}

## Extension Fuzzing

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```
{% endcode %}

## Recursive Fuzzing

When scanning recursively, it automatically starts another scan under any newly identified directories that may have on their pages until it has fuzzed the main website and all of its sub-directories.

Some websites may have a big tree of sub-directories, like `/login/user/content/uploads/...etc`, and this will expand the scanning tree and may take a very long time to scan them all. This is why it is always advised to specify a `depth` to our recursive scan, such that it will not scan directories that are deeper than that depth. Once we fuzz the first directories, we can then pick the most interesting directories and run another scan to direct our scan better.

In Ffuf, we can enable recursive scanning with the `-recursion` flag, and we can specify the depth with the `-recursion-depth` flag. If we specify `-recursion-depth 1`, it will only fuzz the main directories and their direct sub-directories. If any sub-sub-directories are identified (like `/login/user`, it will not fuzz them for pages). When using recursion in Ffuf, we can specify our extension with `-e .php`

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```
{% endcode %}

## Sub-domain Fuzzing

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```
{% endcode %}

{% hint style="info" %}
If the scan returns no results it means that there are no `public` sub-domains under the domain, as it does not have a public DNS record.
{% endhint %}

## Vhost Fuzzing

The key difference between VHosts and sub-domains is that a VHost is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be serving two or more different websites.

{% hint style="warning" %}
VHosts may or may not have public DNS records.
{% endhint %}

To scan for VHosts, without manually adding the entire wordlist to our `/etc/hosts`, we will be fuzzing HTTP headers, specifically the `Host:` header. To do that, we can use the `-H` flag to specify a header and will use the `FUZZ` keyword within it, as follows:

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```
{% endcode %}

## Parameter Fuzzing

### GET Requests

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```
{% endcode %}

{% hint style="warning" %}
A standard request will return too many results, filtering out the default response size will be required.

Use the `-fs` flag to do this.
{% endhint %}

### POST Request

To fuzz the `data` field with Ffuf, we can use the `-d` flag, as we saw previously in the output of `ffuf -h`. We also have to add `-X POST` to send `POST` requests.

{% code overflow="wrap" %}
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
{% endcode %}

{% hint style="success" %}
## Tip

In PHP, "POST" data "content-type" can only accept "application/x-www-form-urlencoded".

So, we can set that in "ffuf" with "-H 'Content-Type: application/x-www-form-urlencoded'".
{% endhint %}

To test the `POST` request with the `id` parameter. We can do that with `curl`, as follows:

{% code overflow="wrap" %}
```bash
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```
{% endcode %}

## Value Fuzzing

Our command should be fairly similar to the `POST` command we used to fuzz for parameters, but our `FUZZ` keyword should be put where the parameter value would be, and we will use the `ids.txt` wordlist we just created, as follows:

{% code overflow="wrap" %}
```bash
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
{% endcode %}
