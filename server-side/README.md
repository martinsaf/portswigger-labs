# Server-side Vulnerabilities

This module covers my documentation of the PortSwigger Web security Academy's **Server-side vulnerabilities** path (labs 1-52).

The goal is to understand the root cause of each vulnerability, how
exploitation works, and to formulate detection hypotheses — even before
having the full infrastructure to test them.

## Categories

- [Path Traversal](./path-traversal/) (1 lab)
- [Access Control](./access-control/) (4 labs done)
- Authentication
- Server-Side Request Forgery (SSRF)
- File Upload Vulnerabilities
- OS Command Injection
- SQL Injection

## Approach

For every lab completed:

1. **Exploit** the vulnerability
2. **Document** the exact steps
3. **Hypothesize** how it could be detected (logs, events)
4. **Map** to MITRE ATT&CK

## Note

I don't yet run a public web server in my home SOC lab. The detection
sections are hypotheses for now. When I add one, I'll revisit these labs
and update them with real detection rules.