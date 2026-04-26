# Lab: User role controlled by request parameter

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Access the admin panel at `/admin` and delete user `carlos`.

## 1. What is the vulnerability?
The application relies on a clint-side cookie to determine user privileges. The `admin` cookie is set to `False` for regular users, but the server trusts its value without any server-side validation. This allows privilege escalation by simply modifying the cookie.

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Opened DevTools -> Application -> Cookies
3. Changed the `Admin` cookie from `False` to `True`
4. Navigated to `/admin` - access granted
5. Deleted user `carlos` to solve the lab.

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism]