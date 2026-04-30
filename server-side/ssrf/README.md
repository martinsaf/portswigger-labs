# Server-Side Request Forgery (SSRF)
[OWASP Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)

SSRF vulnerabilities allow an attacker to induce the server to make requests to unintended locations, such as internal services or the server's own loopback interface.

## Common impact
- Access to internal-only services (e.g., admin panels)
- Bypass of access controls
- Potential acess to sensitive data (e.g., internal APIs or metadata services)

## Labs
1. [Basic SSRF against the local server](./lab-09-basic-ssrf/README.md) - Apprentice