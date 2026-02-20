---
icon: ethernet
---

# 3306 - MySQL

## Fingerprinting

The following queries and their output will tell us that we are dealing with **MySQL**:

| Payload            | When to Use                       | Expected Output                                     | Wrong Output                                              |
| ------------------ | --------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| `SELECT @@version` | When we have full query output.   | MySQL Version.                                      | In MSSQL it returns MSSQL version. Error with other DBMS. |
| `SELECT POW(1,1)`  | When we only have numeric output. | `1`                                                 | Error with other DBMS.                                    |
| `SELECT SLEEP(5)`  | Blind/No Output.                  | Delays page response for 5 seconds and returns `0`. | Will not delay response with other DBMS.                  |

## INFORMATION\_SCHEMA Database

To pull data from tables using `UNION SELECT`, we need to properly form our `SELECT` queries. To do so, we need the following information:

* List of databases
* List of tables within each database
* List of columns within each table

With the above information, we can form our `SELECT` statement to dump data from any column in any table within any database inside the DBMS. This is where we can utilise the `INFORMATION_SCHEMA` Database.

The [INFORMATION\_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) database contains metadata about the databases and tables present on the server. _**This database plays a crucial role while exploiting SQL injection vulnerabilities.**_ As this is a different database, we cannot call its tables directly with a `SELECT` statement. If we only specify a table's name for a `SELECT` statement, it will look for tables within the same database.

So, to reference a table present in another DB, we can use the dot ‘`.`’ operator. For example, to `SELECT` a table `users` present in a database named `my_database`, we can use:

```sql
SELECT * FROM my_database.users;
```

Similarly, we can look at tables present in the `INFORMATION_SCHEMA` Database.

## SCHEMATA

The table [SCHEMATA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html) in the `INFORMATION_SCHEMA` database contains information about all databases on the server. It is used to obtain database names so we can then query them. The `SCHEMA_NAME` column contains all the database names currently present.

Let us first test this on a local database to see how the query is used:

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

+--------------------+
| SCHEMA_NAME        |
+--------------------+
| mysql              |
| information_schema |
| performance_schema |
| ilfreight          |
| dev                |
+--------------------+
6 rows in set (0.01 sec)
```

We see the `ilfreight` and `dev` databases.

{% hint style="info" %}
## Note

The first three databases are default MySQL databases and are present on any server, so we usually ignore them during DB enumeration. Sometimes there's a fourth 'sys' DB as well.
{% endhint %}

Now, let's do the same using a `UNION` SQL injection, with the following payload:

```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- 
```

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Once again, we see two databases, `ilfreight` and `dev`, apart from the default ones. Let us find out which database the web application is running to retrieve ports data from. We can find the current database with the `SELECT database()` query. We can do this similarly to how we found the DBMS version in the previous section:

```sql
cn' UNION select 1,database(),2,3-- 
```

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

We see that the database name is `ilfreight`. However, the other database (`dev`) looks interesting. So, let us try to retrieve the tables from it.

## TABLES

Before dumping data from the `dev` database, we need to get a list of the tables to query them with a `SELECT` statement. To find all tables within a database, we can use the `TABLES` table in the `INFORMATION_SCHEMA` Database.

The [TABLES](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html) table contains information about all tables throughout the database. This table contains multiple columns, but we are interested in the `TABLE_SCHEMA` and `TABLE_NAME` columns. The `TABLE_NAME` column stores table names, while the `TABLE_SCHEMA` column points to the database each table belongs to. This can be done similarly to how we found the database names. For example, we can use the following payload to find the tables within the `dev` database:

```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

{% hint style="info" %}
Note how we replaced the numbers '2' and '3' with 'TABLE\_NAME' and 'TABLE\_SCHEMA', to get the output of both columns in the same query.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
## Note

We added a (`where table_schema='dev'`) condition to only return tables from the 'dev' database, otherwise we would get all tables in all databases, which can be many.
{% endhint %}

We see four tables in the dev database, namely `credentials`, `framework`, `pages`, and `posts`. For example, the `credentials` table could contain sensitive information to look into it.

## COLUMNS

To dump the data of the `credentials` table, we first need to find the column names in the table, which can be found in the `COLUMNS` table in the `INFORMATION_SCHEMA` database. The [COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html) table contains information about all columns present in all the databases. This helps us find the column names to query a table for. The `COLUMN_NAME`, `TABLE_NAME`, and `TABLE_SCHEMA` columns can be used to achieve this. As we did before, let us try this payload to find the column names in the `credentials` table:

```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

The table has two columns named `username` and `password`. We can use this information and dump data from the table.
