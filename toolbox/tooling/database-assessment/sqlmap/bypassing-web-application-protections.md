# Bypassing Web Application Protections

While automated exploitation typically proceeds without issue in ideal testing environments, problems can arise when running tools like SQLMap against targets protected by various security solutions, such as Web Application Firewalls (WAFs) or Intrusion Prevention Systems (IPSs). Fortunately, SQLMap integrates numerous mechanisms and advanced options specifically designed to help bypass these defensive countermeasures.

***

## Bypassing Request Integrity Checks

Modern web applications frequently implement checks to ensure the integrity and expected flow of user requests, often relying on unique tokens or calculated values.

### Anti-CSRF Token Bypass

A primary defence against automation tools involves incorporating anti-Cross-Site Request Forgery (CSRF) tokens into all HTTP requests, particularly those resulting from web-form submissions,. These tokens are essential security features that inadvertently harden applications against unwanted automation.

SQLMap can bypass this mechanism using the `--csrf-token` switch. By specifying the token parameter name, SQLMap automatically attempts to parse the target's response content to retrieve a fresh, valid token value for use in the subsequent request.

If one of the request parameters already contains common token infixes (e.g., `csrf`, `xsrf`, or `token`), SQLMap will prompt the user for confirmation on automatically updating this value for future requests:

#### Scenario: Automatic CSRF Detection

{% code overflow="wrap" %}
```bash
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --cs
```
{% endcode %}

```bash
<SNIP>
POST parameter 'csrf-token' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further
```

Unique Value BypassSome applications enforce unique values within predefined parameters, functioning similarly to anti-CSRF protection but without requiring page content parsing. To handle this, the `--randomize` option should be used, pointing to the parameter name whose value needs to be automatically generated and randomized before being sent in each request.

Calculated Parameter BypassIn scenarios where a web application expects a parameter value to be calculated based on another value (e.g., verifying a hash like `h=MD5(id)`), the `--eval` option can be used. This option evaluates custom Python code immediately before the request is transmitted to the target, allowing the payload value to be correctly generated on the fly.

#### Example: Calculating a Message Digest Hash

{% code overflow="wrap" %}
```bash
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib
```
{% endcode %}

***

## Concealing Identity and Origin

Connection problems can arise if the attacking host or tool is identified and subsequently blacklisted by the target's protective measures.

### User-Agent Blacklisting

If SQLMap frequently encounters `[CRITICAL] connection timed` errors or HTTP 5XX responses, this often indicates that the tool's default `User-Agent` header (e.g., `User-agent: sqlmap/1.4.9...`) is being blacklisted by a WAF or IPS.

To bypass this restriction, the --random-agent switch should be employed, instructing SQLMap to use a randomly chosen browser User-Agent value instead of its default signature. Alternatively, the `--mobile` switch can be used to imitate a smartphone User-Agent.

IP Address ConcealingTo conceal the true source IP address or to circumvent IP blacklisting, traffic can be routed through alternative networks:

* **Proxying:** A working proxy can be set using the `--proxy` option (e.g., `--proxy="socks4://177.39.187.70:33283"`). If multiple proxies are available, the `--proxy-file` option can supply a list that SQLMap cycles through, skipping any that cause problems or are blacklisted.
* **Tor Network:** The `--tor` switch enables the easy use of the Tor network, automatically trying to connect via a local SOCKS4 proxy service (typically port 9050 or 9150). To ensure Tor is functioning correctly, the `--check-tor` switch can be used, which verifies connectivity against the Tor project check service.

***

## Tamper Scripts for WAF/IPS Evasion

One of the most effective methods within SQLMap for bypassing WAF/IPS solutions is the use of "tamper" scripts.

### Operation and Chaining

Tamper scripts are specialised Python scripts that modify requests immediately before transmission, altering the payload content to evade detection.

For instance, a primitive WAF might filter SQL operators like `>` or `=`. The `between` tamper script modifies the payload to replace the greater than operator (`>`) with `NOT BETWEEN 0 AND #` and the equals operator (`=`) with `BETWEEN # AND #`, bypassing these simple textual filters.

Multiple tamper scripts can be combined and executed sequentially using the `--tamper` option (e.g., `--tamper=between,randomcase`). These scripts execute based on a predefined priority to maintain SQL syntax integrity, as some scripts modify the syntax itself while others modify inner content.

{% hint style="danger" %}
#### Warning

Blindly using or combining multiple tamper scripts is generally considered poor practice. It is advised that users first manually assess the target and understand the required bypass method before employing tamper scripts.
{% endhint %}

### Notable Tamper Scripts

<table><thead><tr><th width="150">Tamper-Script</th><th>Description</th></tr></thead><tbody><tr><td><code>0eunion</code></td><td>Replaces instances of <code>UNION</code> with <code>e0UNION</code>.</td></tr><tr><td><code>base64encode</code></td><td>Base64-encodes all characters in a given payload.</td></tr><tr><td><code>between</code></td><td>Replaces the greater than operator (<code>></code>) with <code>NOT BETWEEN 0 AND #</code> and the equals operator (<code>=</code>) with <code>BETWEEN # AND #</code>,.</td></tr><tr><td><code>commalesslimit</code></td><td>Replaces (MySQL) instances like <code>LIMIT M, N</code> with <code>LIMIT N OFFSET M</code> counterpart.</td></tr><tr><td><code>equaltolike</code></td><td>Replaces all occurrences of the operator equals (<code>=</code>) with the <code>LIKE</code> counterpart.</td></tr><tr><td><code>randomcase</code></td><td>Replaces each keyword character with a random case value (e.g., <code>SELECT</code> becomes <code>SEleCt</code>).</td></tr><tr><td><code>space2comment</code></td><td>Replaces the space character ( ) with comments (<code>/**/</code>).</td></tr><tr><td><code>space2dash</code></td><td>Replaces the space character ( ) with a dash comment (<code>--</code>) followed by a random string and a new line (<code>\n</code>).</td></tr><tr><td><code>space2hash</code></td><td>Replaces (MySQL) instances of the space character ( ) with a pound character (<code>#</code>) followed by a random string and a new line (<code>\n</code>).</td></tr><tr><td><code>symboliclogical</code></td><td>Replaces <code>AND</code> and <code>OR</code> logical operators with their symbolic counterparts (<code>&#x26;&#x26;</code> and `</td></tr></tbody></table>

To obtain a complete list of all implemented tamper scripts along with their descriptions, the `--list-tampers` switch can be used.

***

## Miscellaneous Evasion Techniques

### Chunked Transfer Encoding

By employing the `--chunked` switch, the POST request body is split into smaller sections ("chunks"). This technique is useful because blacklisted SQL keywords or signatures can be fragmented across different chunks, allowing the modified request to pass unnoticed by the defence mechanism.

### HTTP Parameter Pollution (HPP)

HTTP Parameter Pollution involves splitting the payload content across multiple values assigned to the same parameter name within the request URL (e.g., `?id=1&id=UNION&id=SELECT...`). This technique relies on the target platform (such as ASP applications) concatenating these multiple values on the server side before they are processed by the vulnerable application logic.
