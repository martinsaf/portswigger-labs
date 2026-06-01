# Cross-Site Scripting (XSS)

[OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

Cross-site scripting vulnerabilities allow an attacker to inject malicious JavaScript into a web application, which then executes in the context of another user's browser. This can lead to session hijacking, credential theft, defacement, and more.

## Types of XSS

| Type | Description |
| - | - |
| **Reflected XSS** | Malicious payload is reflected immediately in the response (e.g., Via seach parameter) |
| **Stored XSS** | Payload is stored on the server (e.g., in a comment or profile) and executed when other users view it |
| **DOM-based XSS** | Payload executes entiry on the client-side without the server reflecting it in the HTML |

## Common impact

- Session hijacking (stealing cookies/tokens)
- Credential theft (keylogging, phishing)
- Defacement of the application
- Bypass of CSRF protections
- Account takeover

## Labs

1. [Reflected XSS into HTML context with nothing encoded](./lab-01-reflected-xss-simple/README.md) - Apprentice
2. [Stored XSS into HTML context with nothing encoded](./lab-02-stored-xss-simple/README.md) - Apprentice 