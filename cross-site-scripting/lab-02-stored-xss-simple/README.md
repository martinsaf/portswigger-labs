# Lab: Stored XSS into HTML context with nothing encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Perform a stored cross-site scripting attack that calls the `alert()` function.

## 0. Attacker mindset

The comment functionality allows users to post comments that are stored on the server and displayed to other users. If the application does not encode or filter HTML special characters, an attacker can inject JavaScript that will execute in the browsers of any user who views the comment page.

## 1. What is the vulnerability?

The application stores user-supplied comments and displays them without any HTML encoding. The comment fields (name, website, and comment body) are all potential injection points. The application does not strip or escape `<script>` tags or other HTML special characters like `<` and `>`.

## 2. How did I exploit it?

1. Navigated to a blog post that allows comments
2. Filled in the comment form with:
   - Name: any value
   - Comment: `<script>alert("You got hacked!")</script>`
   - Email: any valid email
   - Website: optional
3. Submitted the comment
4. When the page reloaded (or when any user viewed the post), the JavaScript executed and the alert popup appeared
5. Lab solved

**Payload used:**
```html
<script>alert("You got hacked!")</script>
```

## 3. Impact

- Persistent JavaScript execution for every user who views the affected page
- Session hijacking (stealing cookies of other users)
- Defacement of the application
- Credential theft via fake login forms
- Widespread impact - stored XSS affects multiple users, not just the attacker

## 4. How can it be fixed?

- HTML-encode all user-supplied content before displaying it (convert < to &lt;, > to &gt;, etc.)
- Use a Content Security Policy (CSP) to restrict script execution
- Validate and sanitize input on the server-side
- Use context-aware output encoding based on where the data is placed (HTML body, attributes, JavaScript, etc.)

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)