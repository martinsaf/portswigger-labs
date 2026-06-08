# Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Break out of a JavaScript string and call the `alert()` function.

## 0. Attacker mindset

When user input is reflected **inside a JavaScript string** (e.g., between `''` or `""`), angle brackets are often HTML-encoded, but that doesn't matter. We're not injecting HTML tags - we're injecting JavaScript code. The goal is to **break out of the string literal** by closing the quotes, then add our own JavaScript, and finally comment out or fix any remaining code to avoid syntax errors.

## 1. What is the vulnerability?

The search functionality includes a JavaScript block that handles search term tracking:

```javascript
<script>
    var searchTerms = 'USER_INPUT';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

The user input from the `search` parameter is placed directly inside the single quotes of the `searchTerms` variable. Angle brackets are HTML-encoded (`<` → `&lt;`, `>` → `&gt;`), but **quotes are not encoded** within the JavaScript context.

## 2. How did I exploit it?

### Discovery

Tested a normal search and viewed the page source. Found the JavaScript block with my input inside the string.

### Code Analysis

The vulnerable code structure:
```javascript
var searchTerms = '[MY INPUT]';
```

Whatever I put inside the quotes becomes part of the JavaScript string.

### Failed Attempt (Conceptual)

Trying `<script>alert(1)</script>` doesn't work because:
1. `<` and `>` become `&lt;` and `&gt;` - not executable HTML
2. Even if they weren't encoded, we're inside a JavaScript string, not HTML context

### The Breakthrough

We need to:
1. **Close the string** with a single quote `'`
2. **Terminate the statement** with a semicolon `;`
3. **Add our JavaScript payload** `alert(1)`
4. **Comment out the rest** with `//` to prevent syntax errors from the remaining code

**Final payload:**
```javascript
';alert(1);//
```

**How it transforms the code:**
```javascript
var searchTerms = '';alert(1);//';
```

**Breakdown:**
| Part | What it does |
|------|---------------|
| `'` | Closes the opening quote of the string |
| `;` | Ends the `var searchTerms = ''` statement |
| `alert(1);` | Our injected JavaScript - executes! |
| `//` | Comments out the rest of the line (`';` becomes ignored) |

### Alternative Solution (Burp Suite)

Another valid payload using JavaScript's loose typing:
```javascript
'-alert(1)-'
```

**Result:**
```javascript
var searchTerms = ''-alert(1)-'';
```

This works because:
- The first `'` closes the string
- `-alert(1)-` subtracts the result of `alert(1)` (which returns `undefined`) from an empty string (coerced to `0`)
- The `-''` at the end is just more arithmetic
- The entire expression is valid JavaScript, though less readable

### Why my payload works better:

`';alert(1);//` is cleaner because:
- Clearly separates the injection from the original code
- The `//` comment prevents any unintended side effects
- Works regardless of what comes after the vulnerable string

## 3. Impact

- Arbitrary JavaScript execution within the page context
- No HTML tags needed - bypasses angle bracket encoding
- Can steal cookies, session tokens, or perform any action the user can
- The payload executes immediately when the script runs (no user interaction required)

## 4. How can it be fixed?

- **Never put user input directly into JavaScript strings**
- Use `JSON.stringify()` to safely encode data for JavaScript
- Escape quotes and backslashes: `'` → `\'`, `"` → `\"`, `\` → `\\`
- Use `encodeURIComponent()` only for URLs, not for JavaScript context
- Consider using `textContent` or safe DOM APIs instead of inline JavaScript

### Example fix:
```javascript
<script>
    var searchTerms = <?php echo json_encode($_GET['search']); ?>;
    // Or on the client side:
    var searchTerms = document.currentScript.getAttribute('data-search');
</script>
```

---

## Key Insight: JavaScript String Injection vs HTML Injection

| Context | Injection technique | Encoding that blocks it |
|---------|---------------------|-------------------------|
| HTML body | `<script>alert(1)</script>` | `<`, `>` encoding |
| HTML attribute | `" onmouseover="alert(1)"` | Quote encoding |
| JavaScript string | `';alert(1);//` | Quote escaping (`\'`) |

This lab demonstrates that **HTML encoding is not enough** - different contexts require different defenses.

---

## Comparison of Payloads

| Payload | Resulting code | Why it works |
|---------|----------------|--------------|
| `';alert(1);//` | `var searchTerms = '';alert(1);//';` | Clean string termination + comment |
| `'-alert(1)-'` | `var searchTerms = ''-alert(1)-'';` | Relies on type coercion in JS |
| `';alert(1);'` | `var searchTerms = '';alert(1);'';` | Works but leaves an extra empty string |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)