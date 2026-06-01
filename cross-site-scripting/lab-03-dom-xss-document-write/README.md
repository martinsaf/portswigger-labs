# Lab: DOM XSS in document.write sink using source location.search

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Perform a DOM-based XSS attack that calls the `alert()` function.

## 0. Attacker mindset

DOM-based XSS is different from reflected or stored XSS. The vulnerability exists entirely in client-side JavaScript. The server may not see the payload at all. The attacker needs to understand the **flow** of data from a controllable **source** (like `location.search`) to a dangerous **sink** (like `document.write`).

## 1. What is the vulnerability?

The application includes JavaScript that:
1. Reads the `search` parameter from the URL (`location.search`)
2. Passes it to a function called `trackSearch(query)`
3. Writes an `<img>` tag directly to the page using `document.write()`

**The vulnerable code:**
```javascript
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) { trackSearch(query); }
```

The `query` variable is concatenated into the `src` attribute without any sanitization or encoding.

## 2. How did I exploit it?

### Discovery

First, identified the correct parameter (`search`) and confirmed it appeared in the page:
- `/?search=TEST` → string found in two locations:
  - `<h1>0 search results for 'TEST'</h1>` (encoded - safe)
  - `<img src="/resources/images/tracker.gif?searchTerms=TEST">` (potentially dangerous)

### Code Analysis

Used DevTools Sources (Ctrl+Shift+F) to search for "trackSearch" and found the vulnerable JavaScript function.

### Failed Attempts (Learning Process)

| Payload | Result | Why it failed |
|---------|--------|---------------|
| `TEST onmouseover=alert(1)` | Injected but required interaction | `onmouseover` needs user to hover, image was 1x1 invisible |
| `x" onerror=alert(1) x="` | No alert | `tracker.gif` loaded successfully, so `onerror` never fired |
| `//lixo" onerror=alert(1) x="` | No alert | Browser treated as relative path, image still loaded |

### The Breakthrough

Realized I couldn't force an image error. The solution was to **exit the `<img>` tag entirely** and create my own element that triggers automatically.

**Final payload:**
```
"><svg onload=alert(1)>
```

**URL-encoded:**
```
/?search=%22%3E%3Csvg+onload=alert(1)%3E
```

**Why it works:**
- `">` closes the `src` attribute and the `<img>` tag
- `<svg onload=alert(1)>` creates a new `<svg>` element with an `onload` event that fires automatically when the element loads

### Tools Used

- **DevTools Elements** - to inspect how the DOM changed after JavaScript execution
- **DevTools Sources** - to locate the vulnerable code and set breakpoints
- **DevTools Network** - to observe requests made by the browser

## 3. Impact

- Arbitrary JavaScript execution in the victim's browser
- No server-side interaction required - the payload never touches the server
- Can bypass many WAFs because the attack is client-side
- Session hijacking, credential theft, defacement

## 4. How can it be fixed?

- Never use `document.write()` with untrusted data
- Use safe DOM manipulation methods like `textContent` or `innerText` instead of `innerHTML`
- Encode data before inserting into HTML context
- Use `encodeURIComponent()` for URL parameters
- Implement a Content Security Policy (CSP) that restricts `script-src`

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)

