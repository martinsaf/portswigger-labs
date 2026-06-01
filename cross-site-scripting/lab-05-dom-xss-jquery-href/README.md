# Lab: DOM XSS in jQuery anchor href attribute sink using location.search source

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Make the "back" link alert `document.cookie`.

## 0. Attacker mindset

DOM-based XSS can also target attributes like `href`. When an application uses jQuery's `.attr()` to set an `href` based on user-controlled data, an attacker can inject the `javascript:` pseudo-protocol. Unlike `document.write` or `innerHTML` sinks that execute automatically, an `href` sink requires **user interaction** (clicking the link). The attack still qualifies as XSS because the JavaScript executes in the context of the victim's session.

## 1. What is the vulnerability?

The submit feedback page includes a "Back" link. JavaScript uses jQuery to dynamically set its `href` attribute based on the `returnPath` parameter from the URL.

**The vulnerable code:**
```javascript
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

**HTML structure:**
```html
<div class="is-linkback">
    <a id="backLink">Back</a>
</div>
```

The `returnPath` value is taken directly from `location.search` and assigned to the `href` attribute without any sanitization.

## 2. How did I exploit it?

### Discovery

Visited the `/feedback` page. Observed the URL already contained:
```
/feedback?returnPath=/
```

The "Back" link pointed to the homepage.

### Code Analysis

Used DevTools to inspect the page source or search for `backLink` to locate the vulnerable jQuery code.

### The Breakthrough

Instead of a normal path like `/`, the `href` attribute can accept the `javascript:` protocol, which executes JavaScript when clicked.

**First attempt:**
```
?returnPath=javascript:document.cookie
```
Clicked "Back" - nothing visible happened. `document.cookie` returns a value but doesn't display it.

**Final payload:**
```
?returnPath=javascript:alert(document.cookie)
```

Clicked "Back" - an alert box appeared showing the cookie value. Lab solved.

## 3. Impact

- JavaScript execution when the victim clicks the malicious link
- Session hijacking via `document.cookie` (stealing session tokens)
- Can be combined with social engineering to trick victims into clicking
- The injected code runs in the context of the vulnerable page, with access to all session data

## 4. How can it be fixed?

- Never set `href` attributes directly from user-controlled data
- Validate that `returnPath` is a relative path (starts with `/`) and contains no `:` or `javascript` substring
- Use a whitelist of allowed paths instead of accepting arbitrary input
- Implement a Content Security Policy (CSP) that blocks `javascript:` pseudo-protocol in links

---

## Key Differences Between Sinks So Far

| Aspect | Lab 3 | Lab 4 | Lab 5 |
|--------|-------|-------|-------|
| Sink | `document.write()` | `innerHTML` | `href` (jQuery `.attr()`) |
| Context | Inside `<img src="...">` | Inside `<span>` tag | Anchor link attribute |
| User interaction needed? | No (onload event) | No (onload event) | **Yes (click required)** |
| Technique | Close tag + inject `<svg onload>` | Direct `<svg onload>` injection | `javascript:alert()` protocol |
| Library used | Vanilla JS | Vanilla JS | jQuery |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)