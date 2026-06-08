# Lab: Reflected XSS into attribute with angle brackets HTML-encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Apprentice\
**Objective:** Perform a cross-site scripting attack that injects an attribute and calls the `alert()` function.

## 0. Attacker mindset

The application HTML-encodes angle brackets (`<` becomes `&lt;`, `>` becomes `&gt;`). This prevents classic `<script>` or `<svg onload>` injection. However, if user input is reflected inside an HTML **attribute** (like `value=""` or `placeholder=""`), the attacker can break out of the attribute context by closing the quotes and injecting a new event handler.

The goal is to inject something like:
```html
" onmouseover="alert(1)
```

## 1. What is the vulnerability?

The search functionality reflects the user's input inside a `value` attribute of an `<input>` tag:

```html
<input type="text" placeholder="Search the blog..." value="USER_INPUT">
```

Angle brackets are encoded:
- `<` → `&lt;`
- `>` → `&gt;`

But **quotes are not encoded**. This allows an attacker to:
1. Close the `value` attribute with `"`
2. Inject a new attribute (e.g., `onmouseover`)
3. Add JavaScript that executes when the user hovers over the input field

## 2. How did I exploit it?

### Discovery

Tested the search bar with a simple payload containing angle brackets:
```
<script>alert(1)</script>
```

Inspected the HTML response and saw:
```html
<input type="text" placeholder="Search the blog..." value="&lt;script&gt;alert(1)&lt;/script&gt;">
```

Angle brackets were encoded, so `<script>` injection was blocked.

### Code Analysis

Used DevTools to inspect the `<input>` element. Noticed the `value` attribute reflected my input exactly as a string, but the quotes around the attribute remained intact.

### The Breakthrough

Since the input is inside a `value` attribute, we can:
- Close the `value` attribute with a double quote
- Inject an event handler attribute
- Close the input tag (optional, event handlers work without closing the tag)

**Final payload:**
```html
" onmouseover="alert(1)
```

**How it renders in HTML:**
```html
<input type="text" placeholder="Search the blog..." value="" onmouseover="alert(1)">
```

**Why it works:**
- The first `"` closes the `value` attribute
- `onmouseover="alert(1)"` creates a new attribute that executes JavaScript when the user hovers over the search box
- The remaining `">` are ignored or close the input tag gracefully

### Testing

1. Entered `" onmouseover="alert(1)` in the search bar
2. Submitted the search
3. Hovered the mouse over the search input field
4. Alert popup appeared → Lab solved

## 3. Impact

- Arbitrary JavaScript execution when the victim **hovers** over the search box
- No `<script>` tag needed - bypasses angle bracket encoding
- Can be combined with social engineering (e.g., "hover here to see results")
- Leads to session hijacking, credential theft, or defacement

## 4. How can it be fixed?

- **Encode quotes** as well (`"` → `&quot;`, `'` → `&#39;`)
- Use context-aware encoding based on where the data is placed (HTML body vs. attribute)
- Implement a strict Content Security Policy (CSP) that restricts `unsafe-inline` event handlers
- Consider using `textContent` instead of setting attributes with untrusted data

---

## Key Insight

| Encoding applied | Can we inject? | Technique |
|-----------------|----------------|-----------|
| Angle brackets only (`<`, `>`) | ✅ Yes | Break out of attribute with `"` or `'` |
| Quotes also encoded | ❌ No | Need a different sink (e.g., URL param in JavaScript) |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)