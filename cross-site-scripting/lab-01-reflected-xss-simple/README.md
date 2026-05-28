# Lab: Reflected XSS into HTML context with nothing encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Perform a cross-site scripting attack that calls the `alert()` function.

## 0. Attacker mindset

The search functionality refelcts user input directly into the HTML response. If the applicatio does nto encode or filter special characters like `<` and `>`, and attacker can inject arbitrary HTML and JavaScript. The goal is to inject a `<script>` tag that executes JavaScript in the victim's browser.

## 1. What is the vulnerability?

The application takes the `search` parameter from the query string and inserts it into the response HTML without any encoding or sanitization. This allows an attacker to break out of the intended context and inject a `<script>` tag.

**Example request:** `GET /?search=<scrip>alert(1)</script>`

**Response (simplified):**
```html
<section class="blog-header">
    <h1>0 search results for `<scritp>alert(1)</script>`</h1>
</section>
```

The browser interprets `<script>alert(1)</script>` as executable JavaScript.

## 2. How did I exploit it?

1. Located the search bar on the main page
2. Entered the following payload: 
```html
<script>alert("You have been HACKED!")</script>
```
3. Submitted the search
4. An alert popup appeared, confirming JavaScript execution
5. Lab solved

## 3. Impact

- Arbitrary JavaScript execution in the victim's browser
- Session hijacking (stealing cookies)
- Defacement or phishing attacks
- Redirection to malicious sites

## 4. How can it be fixed?

- HTML-encode user input before reflecting it. Convert `<` to `&lt`, `>`, `&gt`, etc.
- Use a context-aware encoding library (e.g., OWASP Java Encoder)
- Implement a Content Security Policy (CSP) to restict script execution

---

**MITRE ATT&CK:** T1189 - (Drive-by Compromise)[https://attack.mitre.org/techniques/T1189/]