# SQL Injection (SQLi)

SQL injection allows an attacker to interfere with the queries an application makes to its database. It can lead to unauthorized data access, data modification, and in some cases, full server compromise.

This module covers the **Practitioner** learning path, focusing on UNION attacks, blind SQL injection, error-based techniques, time delays, out-of-band (OAST) exploitation, and database enumeration. 

## Common impact

- Extraction of sensitive data (user credentials, personal information, internal business data)
- Bypass of authentication and access controls
- Data manipulation or deletion
- Remote code execution on the database server in advanced scenarios

## Labs

1. [Retrieve hidden data](/server-side/sql-injection/lab-14-retrieve-hidden-data/README.md) - Apprentice
2. [Login bypass](/server-side/sql-injection/lab-15-login-bypass/README.md) - Apprentice
3. [UNION attack, determining the number of columns](./lab-01-union-columns/README.md) - Practitioner
4. [SQL injection UNION attack, finding a column containing text](./lab-02-find-text-column/README.md) - Practitioner
    - [Burp Suite Notes - SQL injection UNION attack, finding a column containing text](./lab-02-find-text-column/burp-notes.md) - Practitioner
5. [SQL injection attack, querying the database type and version on MySQL and Microsoft](./lab-05-database-version/README.md) - Practitioner
6. [SQL injection attack, listing the database contents on non-Oracle databases](./lab-06-listing-database-contents/README.md) - Practitioner
7. [Blind SQL injection with conditional responses](./lab-07-blind-conditional/README.md) - Practitioner
8. [Blind SQL injection with conditional errors](./lab-08-conditional-errors/README.md) - Practitioner
9. [Visible error-based SQL injection](./lab-09-visible-error/README.md) - Practitioner
10. [Blind SQL injection with time delays and information retrieval](./lab-10-time-based/README.md) - Practitioner
11. [Blind SQL injection with time delays and information retrieval](./lab-11-out-of-band-interaction/README.md) - Practitioner - NOT DONE
12. [SQL injection with filter bypass via XML encoding](./lab-12-xml-encoding-bypass/README.md) - Practitioner
13. [SQL injection attack, querying the database type and version on Oracle](./lab-13-database-version-oracle/README.md) - Practitioner

## References

[OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

[SQL injection cheat sheet from PortSwigger Academy](https://portswigger.net/web-security/sql-injection/cheat-sheet)