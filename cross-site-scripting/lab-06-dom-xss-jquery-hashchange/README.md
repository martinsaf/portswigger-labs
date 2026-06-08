# Lab: DOM XSS in jQuery selector sink using a hashchange event

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Deliver an exploit to the victim that calls the `print()` function in their browser.

## 0. Attacker mindset

DOM-based XSS can use jQuery selectors as sinks. When an application uses `$()` with user-controlled data that is concatenated into a selector string, an attacker can inject HTML. jQuery's `$()` function, when it receives a string starting with `<`, interprets it as HTML and creates new DOM elements. The challenge is triggering the injection at the right time, as the `hashchange` event only fires when the hash changes **after** the page loads.

## 1. What is the vulnerability?

The home page contains JavaScript that listens for the `hashchange` event:

```javascript
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});
```

- **Source:** `window.location.hash` (the fragment identifier after `#`)
- **Sink:** jQuery `$()` selector with `:contains()`
- The hash value is decoded and concatenated directly into the selector string
- No sanitization is applied

## 2. How did I exploit it?

### Understanding the Vulnerability

The vulnerability exists because:
1. The hash is decoded and concatenated directly into the jQuery selector
2. jQuery's `$()` function interprets strings starting with `<` as HTML, not selectors
3. The `hashchange` event fires when the hash **changes after page load**

### The Exploit Technique

The key insight is that the `hashchange` event **only fires when the hash changes after page load**. Direct navigation with a malicious hash doesn't trigger it.

**Solution:** Use an iframe that loads with an innocent hash, then modifies it in the `onload` event to trigger `hashchange`.

### The Working Payload

```html
<iframe src="https://LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

### Step-by-step execution:

1. Iframe loads with `src=".../#"` (innocent hash, no `hashchange` event on load)

2. Iframe's `onload` event fires → executes `this.src += '<img src=x onerror=print()>'`

3. The `src` attribute is modified, which changes the hash to `#<img src=x onerror=print()>`

4. The `hashchange` event fires inside the iframe

5. The vulnerable code executes:
   - `decodeURIComponent(window.location.hash.slice(1))` decodes the hash to `<img src=x onerror=print()>`
   - This is concatenated into the selector: `$('section.blog-list h2:contains(<img src=x onerror=print()>)')`

6. jQuery sees the string starts with `<` and interprets it as **HTML** instead of a selector

7. An `<img src=x onerror=print()>` element is created

8. The image fails to load (`src=x` doesn't exist) → `onerror` event fires → `print()` executes

### Why this approach works:

| Component | Purpose |
|-----------|---------|
| `src="#"` | Initial hash with no payload - `hashchange` doesn't fire on load |
| `onload` attribute | Fires after iframe loads, allowing hash modification |
| `this.src += '...'` | Appends payload, changing the hash and triggering `hashchange` |
| `<img src=x onerror=print()>` | Injected as HTML by jQuery; fails to load, executes `onerror` |

## 3. Key Insights

1. **Event Timing:** The `hashchange` event only fires when the hash **changes after page load**. Direct navigation with a malicious hash in the URL doesn't work.

2. **jQuery HTML Creation:** When jQuery's `$()` receives a string starting with `<`, it switches from CSS selector parsing to HTML creation mode, allowing XSS injection.

3. **Iframe Technique:** Using an iframe with an `onload` handler allows us to modify the hash after the page loads, triggering the vulnerable `hashchange` event.

## 4. Impact

- Arbitrary JavaScript execution via `print()` (or `alert()` for proof of concept)
- Can be combined with social engineering (luring victim to click a link)
- The iframe technique allows silent exploitation without user interaction beyond visiting the attacker's page

## 5. How can it be fixed?

- Never concatenate user input into jQuery selectors
- Use `text()` or attribute selectors with proper escaping instead of `:contains()`
- Sanitize or encode data before using it with `$()`
- Consider using `find()` with safe parameters instead of dynamic selector strings
- Implement a Content Security Policy (CSP)

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)