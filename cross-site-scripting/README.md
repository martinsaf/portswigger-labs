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
3. [DOM XSS in document.write sink using source location.search](./lab-03-dom-xss-document-write/README.md) - Apprentice
4. [DOM XSS in innerHTML sink using source location.search](./lab-04-dom-xss-innerhtml/README.md) - Apprentice 
5. [DOM XSS in jQuery anchor href attribute sink using location.search source](./lab-05-dom-xss-jquery-href/README.md) - Apprentice
6. [DOM XSS in jQuery selector sink using a hashchange event](./lab-06-dom-xss-jquery-hashchange/README.md) - Apprentice
7. [Reflected XSS into attribute with angle brackets HTML-encoded](./lab-07-reflected-xss-attribute-encoded/README.md) - Apprentice
8. [Stored XSS into anchor href attribute with double quotes HTML-encoded](./lab-08-stored-xss-href-javascript-protocol/README.md) - Apprentice
9. [Reflected XSS into a JavaScript string with angle brackets HTML encoded](./lab-09-reflected-xss-js-string/README.md) - Apprentice
10. [DOM XSS in document.write sink using source location.search inside a select element](./lab-10-dom-xss-document-write-select-element/README.md) - Practitioner
11. [DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded](./lab-11-dom-xss-angularjs-expression/README.md) - Practitioner
12. [Reflected DOM XSS](./lab-12-reflected-dom-xss/README.md) - Practitioner