# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Display the database version string

## 0. Attacker mindset

The lab title hints at MySQL or Microsoft SQL Server. The hint in the page source shows the expected version string: `8.0.42-0ubuntu0.20.04.1`, which looks like MySQL on Ubuntu. I need to extract the version using a UNION attack.

## 1. What is the vulnerability?

The product category filter passes user input directly into a SQL query without sanitization. An attacker can inject a `UNION SELECT` to execute arbitrary queries and display the results

## 2. How did I exploit it?

1. Confirmed the injection point with `Lifestyle''--`, which returned 200 OK.
2. Attempted `ORDER BY` and `UNION SELECT` with `--` comments; all returned HTTP 500. The `--` comment syntax requires a trailing space, which Burp Repeater was not sending correctly in this lab.
3. Switched to `#`, the alternative MySQL comment character: `Accessories'+UNION+SELECT+'abc','def'#`. Returned 200 OK with `abc` and `def` in the response. Confirmed 2 columns, both accepting text.
4. Injected `@@version` into the first colum: `Accessories'+UNION+SELECT+@@version,NULL#`. The response displayed `8.0.42-0ubuntu0.20.04.1`, matching the hint and solving the lab.

## 3. Impact

- Attacker learns the exact database type and version
- Version information enables targeted exploitation of known vulnerabilities
- Confirms the injection point can extract arbitrary data from the database

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all database access
- Validate user input against an allowlist of expected values
- Do not rely on blacklisting SQL keywords or comment characters