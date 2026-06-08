# Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Submit a comment that calls the `alert()` function when the comment author name is clicked.

## 0. Attacker mindset

Stored XSS doesn't always require breaking out of HTML tags or using `<script>`. When an application stores user input and later places it inside an HTML **attribute** like `href`, an attacker can use the `javascript:` pseudo-protocol. Even if double quotes are HTML-encoded, the `href` attribute accepts `javascript:alert(1)` directly without quotes.

The challenge is identifying **which field** ends up in which HTML context.

## 1. What is the vulnerability?

The comment functionality stores multiple fields:
- `comment` (the message body)
- `name` (display name)
- `email` (required, but not displayed as a link)
- `website` (optional field)

When a comment is displayed, the `website` field is used as the `href` attribute of an `<a>` tag around the commenter's name:

```html
<a id="author" href="USER_WEBSITE">USER_NAME</a>
```

Double quotes in the input are HTML-encoded (`"` → `&quot;`), but this doesn't matter because:
1. We don't need to break out of quotes to inject into `href`
2. The `javascript:` protocol doesn't require quotes

## 2. How did I exploit it?

### Discovery

First, inspected the comment form:

```html
<form action="/post/comment" method="POST">
    <textarea name="comment"></textarea>
    <input type="text" name="name">
    <input type="email" name="email">
    <input type="text" name="website">
    <button type="submit">Post Comment</button>
</form>
```

Posted a test comment with normal values to see how each field is rendered.

### Code Analysis

After submitting a comment, inspected the rendered HTML:

```html
<section class="comment">
    <a id="author" href="https://example.com">Test User</a>
    <p>Test comment body</p>
</section>
```

The `website` field became the `href` attribute. The `name` field became the link text. The `comment` body was plain text.

### The Breakthrough

The `href` attribute accepts the `javascript:` pseudo-protocol. Instead of a normal URL like `https://example.com`, we can put:

```
javascript:alert(1)
```

This executes JavaScript when the link is **clicked**.

**Final payload in the Website field:**
```
javascript:alert(1)
```

**Other fields used:**
- **Name:** `Lucky Luke` (any name works)
- **Email:** `luckyL@warn.you` (required but irrelevant)
- **Comment:** `Check my profile for content like this.` (optional)

### Resulting HTML:

```html
<a id="author" href="javascript:alert(1)">Lucky Luke</a>
```

### Execution:

1. Posted the comment
2. Any user viewing the blog post sees the comment
3. When they click on `Lucky Luke` (the author name)
4. `javascript:alert(1)` executes → alert popup appears
5. Lab solved

## 3. Impact

- Persistent stored XSS - affects every user who views the comment and clicks the link
- Requires user interaction (clicking the author name)
- Can be combined with social engineering ("Click my profile for a surprise")
- Leads to:
  - Session hijacking via `javascript:alert(document.cookie)`
  - Redirect to malicious sites
  - Credential theft with fake login forms

## 4. How can it be fixed?

- **Never allow `javascript:` protocol** in `href` attributes from user input
- Validate the `website` field to only accept HTTP/HTTPS URLs with a proper domain
- Use a whitelist of allowed URL schemes (only `http:` and `https:`)
- Sanitize/encode the entire URL before inserting into `href`
- Implement a Content Security Policy (CSP) that blocks `javascript:` pseudo-protocol

### Example fix (server-side validation):
```javascript
function validateUrl(url) {
    const allowed = ['http://', 'https://'];
    return allowed.some(prefix => url.startsWith(prefix));
}
```

---

## Key Insight

| Field | Where it goes | Can inject XSS? | Technique |
|-------|---------------|-----------------|-----------|
| `comment` | Plain text between tags | ❌ (if encoded) | Would need `<script>` or `<img>` |
| `name` | Link text (`<a>text</a>`) | ❌ (not an attribute) | Text is HTML-encoded |
| `website` | `href` attribute | ✅ YES | `javascript:alert(1)` |

The vulnerability exists because the `website` field is **not validated** to ensure it's a real HTTP/HTTPS URL.

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)