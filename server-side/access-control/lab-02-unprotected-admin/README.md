# Lab: Unprotected admin functionality

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Solve the lab by deleting the user `carlos`

## 1. What is the vulnerability?
The application has an admin panel that is intended only for privileged users. However, it does not verify whether the user requesting the page actually has admin rights. The admin functionality is hosted at a guessable or easily discoverable URL (like `/admin`), and is accessible to any authenticated user who navigates directly to it.

In this case, the URL was disclosed in `/robots.txt`. The `robots.txt` file is used by websites to tell search engines which pages not to index. However, it is publicly accessible and can inadvertently reveal sensitive paths like admin panels to anyone who checks it.

## 2. How did I exploit it?

1. Checked `/robots.txt` and found a `Disallow` entry for `/administrator-panel`.
2. Navigated directly to `/administrator-panel` without any authentication.
3. The admin panel was fully accessible. Deleted user `carlos` to solve the lab.

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/)

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)