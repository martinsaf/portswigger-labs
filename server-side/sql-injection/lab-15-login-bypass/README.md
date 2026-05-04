# Lab: SQL injection vulnerability allowing login bypass

**Category:** SQL Injection
**Difficulty:** Apprentice
**Objective:** Log in as the `administrator` user without a password.

## 0. Attacker mindset

Login forms that construct SQL queries by concatenating user input are prime targets for injection. If I can break out of the intended string literal and comment out the password check, the applcation will authenticate me as any user without needing their credentials

## 1. What is the vulnerability?

The login function builds a SQL query by directly embedding the supplied username and password into the `WHERE` clause. By injecting a single quote to close the username string followed by the SQL comment sequence `--`, the rest of the query (the password check) is ignored. The resulting query becomes: `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`. The server authenticates the attacker as `administrator`

## 2. How did I exploit it?

1. Navigated to the login page
2. Entered `administrator'--` as the username
3. Entered any arbitrary value in the password field (the field requires non-empty input, but its value is ignored by the database)
4. Submitted the form. The application logged me in as `administrator`, solving the lab

## 3. Impact

- Complete bypass of authentication
- Full access to the administrator account and its associated privileges 
- Ability to view, modify, or delete sensitive data
- Potential for further compromise of the application and underlying server

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) instead of string concatenation for all SQL operations
- Never trust user input; validate and sanitize data before using it in queries
- Implement proper error handling to avoid leaking database information
- Apply least privilege to database accounts used by web applications

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)