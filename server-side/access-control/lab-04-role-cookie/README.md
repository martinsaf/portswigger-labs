# Lab: User role controlled by request parameter

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Access the admin panel at `/admin` and delete user `carlos`

## 0. Attacker mindset

When the application stores authorization data (like user roles) on the client side, it's inviting manipulation. Cookies, hidden form fields, and URL parameters are all modifiable by the user. If the server trusts these values without verifying them, an attacker can simply edit them to elevate priviliges. The first step after logging in is to inspect what client-side data the application sets and test whether chaning it grants extra access

## 1. What is the vulnerability?

The application relies on a client-side cookie to determine user privileges. The `admin` cookie is set to `False` for regular users, but the server trusts its value without any server-side validation. This allows privilege escalation by simply modifying the cookie.

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Opened DevTools -> Application -> Cookies
3. Changed the `Admin` cookie from `False` to `True`
4. Navigated to `/admin` - access granted
5. Deleted user `carlos` to solve the lab.

## 3. Impact

- Unauthorized privilege escalation to administrator level
- Full access to administrative functions (user deletion, data modification)
- Complete trust in client-supplied data can lead to total application compromise

## 4. How can it be fixed?

- Never rely on client-side data (cookies, hidden fields, URL parameters) for authorization decisions
- Store user roles and permissions on the server, associated with the session identifier
- Validate the user's role on the server for every priviledged request, regardless of what the client sends

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)