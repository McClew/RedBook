# Handling Errors

When setting up or running SQLMap, particularly when dealing with HTTP requests, we may encounter problems. This section details the recommended mechanisms for diagnosing the cause of errors and fixing them appropriately.&#x20;

***

## Error Diagnosis Mechanisms

SQLMap provides several built-in mechanisms to help users understand what is happening during a scan, especially when unexpected behaviour or errors occur.

### Displaying DBMS Errors

If a database management system (DBMS) error is parsed by SQLMap during its execution, enabling the --parse-errors switch forces SQLMap to display this error message as part of the program run. This immediately provides clarity on the potential issue, helping you identify and fix syntax problems or access violations.&#x20;

Example Output (using --parse-errors):&#x20;

```bash
<SNIP>
[16:09:20] [INFO] testing if GET parameter 'id' is dynamic
[16:09:20] [INFO] GET parameter 'id' appears to be dynamic
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE: Syntax error or access violation: 1064 You have an [16:09:20] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE: Syntax error or access violation: 1064 You have 
```

### Increasing Verbosity Level

The `-v` (verbosity) option increases the detail level of the console output. Verbosity levels range from 0 to 6 (default is 1). By setting the verbosity level to 6 (`-v 6`), SQLMap will directly print all errors and the full HTTP requests and responses to the terminal. This allows you to follow along with everything SQLMap is doing in real-time, aiding in detailed debugging.

Example Command and Output (using -v 6):

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

```bash
<SNIP>
[16:17:40] [TRAFFIC OUT] HTTP request [#1]: GET /?id=1 HTTP/1.1
Host: www.example.com
Cache-control: no-cache
Accept-encoding: gzip,deflate
Accept: */*
User-agent: sqlmap/1.4.9 (http://sqlmap.org)
Connection: close
[16:17:40] [TRAFFIC IN] HTTP response [#1] (200 OK):
Date: Thu, 24 Sep 2020 14:17:40 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 914
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://www.example.com:80/?id=1

<!DOCTYPE html> <html lang="en">
<SNIP>
```

### Storing Traffic to a File

Using the `-t` option allows you to store the entire HTTP traffic content (both sent requests and received responses) to a specified local output file. This file can then be examined manually offline to pinpoint issues related to communication, headers, or response content.

Example Command:

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

### Redirecting Traffic via Proxy

For manual, in-depth investigation and interaction, the `--proxy` option can redirect all SQLMap traffic through a Man-in-the-Middle (MiTM) proxy tool, such as Burp Suite. This enables you to manually investigate requests, repeat them, and use the advanced features of the proxy tool to diagnose problems.

{% hint style="info" %}
#### Note

If you are experiencing `[CRITICAL] connection timed` errors frequently, especially if you can browse the site normally, it might be because the default SQLMap `User-Agent` header is being blacklisted by a Web Application Firewall (WAF) or Intrusion Prevention System (IPS). You are advised to use the `--random-agent` switch to automatically change the default User-Agent to a randomly selected browser value, thus bypassing this detection. Alternatively, manually specifying a trusted proxy might also resolve connection issues.
{% endhint %}
