# Lab: User ID controlled by request parameter, with unpredictable user IDs

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Find the GUID for `carlos` and submit his API key to solve the lab.

## 0. Attacker mindset

When user identifiers are unpredictable (GUIDs), direct enumeration is not feasible. However, applications often leak these identifiers in other public areas, such as blog posts, comments, or message forums. The attacker's goal is to map the application's visible content, looking for references to target users that include their GUID. Once found, the same IDOR principle applies: substitute the identifier in the account page URL and check if access is granted.

## 1. What is the vulnerability?

The application identifies users via GUIDs in the `id` parameter on the account page. While GUIDs are not predictable, the application leaks the GUID of `carlos` in public blog posts through the author link. An attacker who finds this GUID can access another user's account and retrieve sensitive data (API key).

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Browsed blog posts on the Home page and searched for `carlos` in the page source of each post.
3. Found a post authored by `carlos` containing a link with his GUID: `/blogs?userId=44ae5afd-5a62-4873-b577-5b7e0266d0b1`
4. Navigated to `/my-account?id=44ae5afd-5a62-4873-b577-5b7e0266d0b1`.
5. Retrieved `carlos` API key and submitted it to solve the lab.

## 3. Impact

- Unauthorized access to another user's account and sensitive data (API key)
- Horizontal privilege escalation via information disclosure
- Leaked GUIDs can be used to systematically harvest user data if multiple GUIDs are exposed in public content

## 4. How can it be fixed?

- Avoid exposing user identifiers (including GUIDs) in public areas of the application; use pseudonyms or generic references where possible
- Enforce server-side authorization checks on every account page request: verify that the logged-in user matches the requested resource
- Do not rely on the unpredictability of GUIDs as the sole layer of access control

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)