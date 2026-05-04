# Server-side Vulnerabilities

This module covers my documentation of the PortSwigger Web security Academy's **Server-side vulnerabilities** path.

**Status:** Completed on 04 May 2026

The goal is to understand the root cause of each vulnerability, how
exploitation works, and to formulate detection hypotheses — even before
having the full infrastructure to test them.

## Categories

- [Path Traversal](./path-traversal/) (1 lab)
- [Access Control](./access-control/) (5 labs done)
- [Authentication](./authentication/) (2 labs)
- [Server-Side Request Forgery (SSRF)](./ssrf/) (2 labs)
- [File Upload Vulnerabilities](./file-upload/) (2 labs)
- [OS Command Injection](./os-command-injection/) (1 labs)
- [SQL Injection](./sql-injection/) (2 labs)

## Approach

For every lab I solve, I document:

0. Attacker mindset
1. What is the vulnerability?
2. How did I exploit it?
3. Impact
4. How can it be fixed?
5. MITRE ATT&CK

## Note

I don't yet run a web server (probably Nginx, maybe Apache) in my home SOC lab. When I add one, I plan to revisit key labs and document the corresponding detection artifacts (logs, SIEM rules).