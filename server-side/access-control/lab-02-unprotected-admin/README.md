# Lab: Unprotected admin functionality

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Solve the lab by deleting the user `carlos`

## 0. Attacker mindset

When an application has a privileged area like an admin panel, the first questions are: "Where is it, and can I reach it without being admin?". Sensitive functionality that is hidden behind a guessable URL or disclosed in publicly accessible files (like `robots.txt`) is a clear sign of poor access control. The attacker's goal is to find that entry point and test whether the server enforces any authorization before granting access.

## 1. What is the vulnerability?
The application has an admin panel that is intended only for privileged users. However, it does not verify whether the user requesting the page actually has admin rights. The admin functionality is hosted at a guessable or easily discoverable URL (like `/admin`), and is accessible to any authenticated user who navigates directly to it.

In this case, the URL was disclosed in `/robots.txt`. The `robots.txt` file is used by websites to tell search engines which pages not to index. However, it is publicly accessible and can inadvertently reveal sensitive paths like admin panels to anyone who checks it.

## 2. How did I exploit it?

1. Checked `/robots.txt` and found a `Disallow` entry for `/administrator-panel`.
2. Navigated directly to `/administrator-panel` without any authentication.
3. The admin panel was fully accessible. Deleted user `carlos` to solve the lab.

## 3. Impact

- Full administrative control of the application without needing valid admin credentials
- Ability to delete user accounts, modify data, or take down services
- Exposure of sensitive functionality to any user who discovers the URL

## 4. How can it be fixed?

- Do not rely on obscurity. The admin interface must always require proper authentication and authorization
- Use a `robots.txt` file only for search engine directives, not as a security measure
- Implement server-side access control checks on every privleged page, not just on the login form

**MITRE ATT&CK:** [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/)

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)