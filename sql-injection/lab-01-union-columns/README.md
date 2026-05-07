# Lab: SQL injection UNION attack, determining the number of columns returned by the query

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Determine the number of columns returned by the query using a UNION attack that returns an additional row of null values.

## 0. Attacker mindset

Before I can extract data from other tables with a UNION attack, I need to know how many columns the original query returns. Two techniques are available: `ORDER BY` with increasing indexes until an error occurs, or `UNION SELECT` with increasing numbers of `NULL` values until the database accepts the query

## 1. What is the vulnerability?

The product category filter is vulnerable to SQL injection. The application builds a `SELECT` query by concatenating the user-supplied `category` parameter without sanitization. An attacker can inject the `UNION SELECT` operator to append additional result rows, provided the number of columns matches the original query

## 2. How did I exploit it?

1. Navigated to a product category page (`/filter?category=...`)
2. Opened DevTools -> Network, located the GET request, and used **Edit and Resend**
3. Tested with `ORDER BY` clauses to determine the number of columns:
    - `' ORDER BY 1--` -> valid
    - `' ORDER BY 2--` -> valid
    - `' ORDER BY 3--` -> valid
    - `' ORDER BY 4--` -> HTTP 500 error
    This indicated the query return **3 columns**
4. Constructed a UNION injection payload with the correct number of NULLs: `' UNION SELECT NULL,NULL,NULL--`
5. Sent the request. The page loaded successfully and displayed an empty row in the product table, confirming the column count and solving the lab.

## 3. Impact

- A confirmed column count is the prerequisite for extracting data from other database tables via UNION injection
- The attacker now knows the exact structure needed for further exploitation (e.g., retrieving usernames, passwords, credit card data)
- Event without extracting data, the confirmed SQL injection vulnerability demonstrates a critical failure in input validation

## 4. How can it be fixed?

- Use paarametrized queries (prepared statements) for all database access, not string concatenation
- If dynamic SQL is unavoidable, validate and sanitize user input against a strict allowlist
- Configure the web application to not display database error message to the user

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

**CWE:** [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

**CAPEC:** [CAPEC-66: SQL Injection](https://capec.mitre.org/data/definitions/66.html)

**OWASP:** [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)

**EMB3D TID-320:** [TID-320: SQL Injection](https://emb3d.mitre.org/threats/TID-320.html)