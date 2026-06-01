# Lab: DOM XSS in innerHTML sink using source location.search

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Perform a DOM-based XSS attack that calls the `alert()` function.

## 0. Attacker mindset

DOM-based XSS can use different sinks. While the previous lab used `document.write()`, this lab uses `innerHTML`. These sinks behave differently. `innerHTML` parses HTML but has a critical limitation: it **does not execute `<script>` tags** inserted via JavaScript. An attacker needs to find alternative HTML elements that trigger JavaScript execution automatically.

## 1. What is the vulnerability?

The application includes JavaScript that:
1. Reads the `search` parameter from the URL (`location.search`)
2. Passes it to a function
3. Assigns the value directly to `innerHTML` of a `<span>` element

**The vulnerable code:**
```javascript
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    doSearchQuery(query);
}
```

The `query` variable is inserted as raw HTML into the page. No sanitization or encoding is applied.

## 2. How did I exploit it?

### Discovery

Tested `/?search=teste` and inspected the DOM. Found the input inside a `<span id="searchMessage">teste</span>`.

### Code Analysis

Used DevTools Sources (Ctrl+Shift+F) to search for `searchMessage` and found the vulnerable JavaScript function.

### Testing HTML Injection

Tested `<b>teste</b>` and the text appeared **bold** - confirmed that `innerHTML` interpreted HTML tags.

### The Script Tag Block

Tested `<script>alert("Gotcha!")</script>` - **failed.** Learned that `innerHTML` does not execute `<script>` tags inserted programmatically as a security measure.

### The Solution

Applied knowledge from the previous lab: use an element with an automatic event instead of `<script>`.

**Final payload:**
```html
<svg onload=alert(1)>
```

**URL-encoded:**
```
/?search=<svg onload=alert(1)>
```

**Why it works:**
- `<svg>` is a valid HTML tag that `innerHTML` accepts
- `onload` fires **automatically** when the SVG element loads
- No user interaction required

## 3. Impact

- Arbitrary JavaScript execution in the victim's browser
- Bypasses the `<script>` tag restriction that some developers mistakenly rely on
- Can lead to session hijacking, credential theft, and account takeover

## 4. How can it be fixed?

- Never assign untrusted data to `innerHTML`
- Use `textContent` instead when inserting plain text
- If HTML is necessary, use a sanitization library like DOMPurify
- Implement a Content Security Policy (CSP) that restricts `script-src` and `unsafe-inline`

---

## Key Difference from Lab 3

| Aspect | Lab 3 (`document.write`) | Lab 4 (`innerHTML`) |
| - | - | - |
| Sink | `document.write()` | `innerHTML` |
| Context | Inside `<img src="...">` | Inside `<span>` tag |
| `<script>` execution | Depends on context | **Does NOT execute** |
| Solution | Close existing tag + inject new element | Direct injection of `<svg onload>` |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)