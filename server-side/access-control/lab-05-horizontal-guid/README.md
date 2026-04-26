# Lab: User ID controlled by request parameter, with unpredictable user IDs

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Find the GUID for `carlos` and submit his API key to solve the lab.

## 1. What is the vulnerability?

The application identifies users via GUIDs in the `id` parameter on the account page. While GUIDs are not predictable, the application leaks the GUID of `carlos` in public blog posts through the author link. An attacker who finds this GUID can access another user's account and retrieve sensitive data (API key).

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Browsed blog posts on the Home page and searched for `carlos` in the page source of each post.
3. Found a post authored by `carlos` containing a link with his GUID: `/blogs?userId=44ae5afd-5a62-4873-b577-5b7e0266d0b1`
4. Navigated to `/my-account?id=44ae5afd-5a62-4873-b577-5b7e0266d0b1`.
5. Retrieved `carlos` API key and submitted it to solve the lab.


## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)