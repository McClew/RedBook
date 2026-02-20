---
icon: ethernet
---

# 1433 - MSSQL

{% hint style="info" %}
## MSSQL Ports

By default, MSSQL uses ports `TCP/1433` and `UDP/1434`, however, when MSSQL operates in a "hidden" mode, it uses the `TCP/2433` port.
{% endhint %}

## Authentication Mechanisms

MSSQL supports two [authentication modes](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server), which means that users can be created in Windows or the SQL Server:

| Windows authentication mode | This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials. |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mixed mode                  | Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.                                                                                                                                                                                     |

