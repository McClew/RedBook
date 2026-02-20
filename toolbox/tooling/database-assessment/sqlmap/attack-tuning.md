# Attack Tuning

In most scenarios, SQLMap should function effectively upon being provided with the target details. However, it offers specific options designed to fine-tune the detection phase by customising SQL injection attempts. Every payload that SQLMap sends is composed of a vector, which carries the useful SQL code, and boundaries, consisting of prefix and suffix formations used to properly inject the vector into the vulnerable SQL statement.

***

## Prefix and Suffix Customisation

In rare situations where the regular SQLMap testing boundary formations are not sufficient, you may need to define custom prefix and suffix values.

* The `--prefix` option specifies a static string inserted before the attack vector.
* The `--suffix` option specifies a static string appended after the attack vector.These options ensure that all vector values are enclosed correctly within the required syntax elements of the target SQL statement.

### Scenario: Custom Boundary Injection

If the underlying vulnerable code structure is `... WHERE id LIKE (('` . `$_GET["q"]` . `')) LIMIT 0,1`, applying the switches `--prefix="%'))" --suffix="-- -"` ensures that the SQL payload is correctly encapsulated, resulting in the following valid SQL statement on the server:

{% code overflow="wrap" %}
```sql
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1
```
{% endcode %}

***

## Adjusting Detection Scope (Level and Risk)

By default, SQLMap utilises a predefined set of boundaries and vectors determined to have a high chance of success. You can expand these sets using the `--level` and `--risk` options.

### Level (`--level`)

The `--level` option (ranging from 1 to 5, with a default of 1) increases the thoroughness of the scan. It extends the number of vectors and boundaries checked, prioritising those with a higher expectation of failure (the lower the expectation of success, the higher the level tested).

### Risk (`--risk`)

The `--risk` option (ranging from 1 to 3, with a default of 1) expands the vector set based on the potential risk of causing issues on the target database, such as accidental data loss or a denial-of-service condition.

Raising both options significantly increases the duration of the scan. For example, testing a single parameter using the default settings (`--level=1 --risk=1`) involves checking up to 72 payloads, whereas using the maximum configuration (`--level=5 --risk=3`) increases this to 7,865 payloads.

{% hint style="warning" %}
#### Note

Increasing the risk level may be required if `OR` payloads are needed, such as when attacking login pages. However, `OR` payloads are considered inherently dangerous as they can potentially interact with underlying vulnerable SQL statements that modify database content (e.g., `DELETE` or `UPDATE`).
{% endhint %}

### Displaying Payloads

You can use the verbosity switch (`-v`) set to 3 or higher to see the exact payloads (`[PAYLOAD]`) being tested when combined with `--level` adjustments:

```bash
sqlmap -u www.example.com/?id=1 -v 3 --level=5
```

```bash
<SNIP>
[14:17:07] [PAYLOAD] 1) AND 5907=7031-- AuiO [7]
[14:17:07] [PAYLOAD] 1')) AND 1049=6686 AND (('OoWT' LIKE 'OoWT [7]
<SNIP>
```

***

## Response Handling and Technique Selection

For fine-tuning detection mechanisms, particularly when dealing with unpredictable or highly dynamic responses, several specialised options are available.

### &#xD;Response-Based Tuning&#xD;

These options help reliably distinguish between TRUE and FALSE responses by targeting specific elements of the HTTP traffic:

<table><thead><tr><th width="122">Option</th><th>Description</th><th>Scenario</th></tr></thead><tbody><tr><td><code>--code</code></td><td>Fixates detection to a specific HTTP status code.</td><td>Used if TRUE responses return a predictable status code (e.g., 200) while FALSE responses return a different code (e.g., 500).</td></tr><tr><td><code>--titles</code></td><td>Bases response comparison only on the content of the HTML <code>&#x3C;title></code> tag.</td><td>Useful if the only detectable difference between responses is in the page title.</td></tr><tr><td><code>--string</code></td><td>Bases detection on the presence of a specific string (e.g., <code>success</code>) found only in TRUE responses.</td><td>This avoids needing advanced comparison techniques when a distinct, constant string is present.</td></tr><tr><td><code>--text-only</code></td><td>Strips all HTML tags (e.g., <code>&#x3C;script></code>, <code>&#x3C;style></code>) and compares responses based solely on the visible textual content,.</td><td>Advisable when dealing with dynamic content from complex HTML structures.</td></tr></tbody></table>

### Injection Technique Control

You can manually specify which injection types SQLMap should use with the `--technique` option. This is useful if specific techniques cause issues (like timeouts) or if you want to focus the scan only on likely successful types.

The available technique flags are:

* B: Boolean-based blind
* E: Error-based
* U: UNION query-based
* T: Time-based blind
* S: Stacked queries

{% hint style="info" %}
#### Skipping Time-Based Queries

To skip time-based blind and stacked queries, testing only the remaining methods:

```bash
sqlmap -u "http://www.example.com/?id=1" --technique=BEU
```
{% endhint %}

### UNION Query Specific Tuning

If UNION query payloads require supplementary information, dedicated switches can be used:

* `--union-cols` \
  Specifies the exact number of columns used in the vulnerable SQL query (e.g., `--union-cols=17`).
* `--union-char` \
  Provides an alternative filler character (e.g., `--union-char='a'`) if the default filler values (`NULL` or random integers) are incompatible with the data types required by the query results.
* `--union-from` \
  Specifies a necessary `FROM <table>` appendix (e.g., `--union-from=users`) required at the end of the `UNION` query for specific database management systems (such as Oracle). This is particularly relevant if the DBMS name could not be successfully detected beforehand.
