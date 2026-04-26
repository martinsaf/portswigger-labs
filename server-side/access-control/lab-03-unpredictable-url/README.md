
# Lab: Unprotected admin functionality with unpredictable URL

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** This lab has an unprotected adminel. It's located at an unpredictable location, but the location is disclosed somewhere in the application. Solve the lab by accessing the admin panel, and using it to delete the user `carlos`.

## 1. What is the vulnerability?
The application hides the admin panel behind a less predictable URL (`/admin-x312yg`, security through obscurity). However, the URL is still exposed in the page's JavaScript code, which is visible to all user regardless of their role. An attacker can find it by inspecting the THML/JS source after logging in.

## 2. How did I exploit it?

1. Viewed the page source of the home page
2. Found a `<script>` block that references the admin panel at `/admin-x312yg`.
3. Navigated directly to `/admin-x312yg`
4. The admin panel loaded without any authentication or access control check. Deleted user `carlos` to solve the lab. 

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*


**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)