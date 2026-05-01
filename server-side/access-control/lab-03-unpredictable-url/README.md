
# Lab: Unprotected admin functionality with unpredictable URL

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application. Solve the lab by accessing the admin panel, and using it to delete the user `carlos`.

## 0. Attacker mindset

Even when a sensitive area is hidden behind an unpredictable URL, the application may still leak its location. Client-side code (HTML, Javascript) is accessible to everyone, regardless of their role. If the admin panel URL appears anywhere in the source, an attacker who inspects the code can find it and attempt to access it directly. Security through obscurity is not security.

## 1. What is the vulnerability?
The application hides the admin panel behind a less predictable URL (`/admin-x312yg`, security through obscurity). However, the URL is still exposed in the page's JavaScript code, which is visible to all users regardless of their role. An attacker can find it by inspecting the HTML/JS source after logging in.

## 2. How did I exploit it?

1. Viewed the page source of the home page
2. Found a `<script>` block that references the admin panel at `/admin-x312yg`.
3. Navigated directly to `/admin-x312yg`
4. The admin panel loaded without any authentication or access control check. Deleted user `carlos` to solve the lab. 

## 3. Impact

- Unauthorized access to the admin panel despite the obscured URL
- Full administrative control without valid credentials
- Ability to delete users, modify data, or disrupt services

## 4. How can it be fixed?

- Do not embed sensitive URLs in client-side code that is accessible to unprivileged users
- Enforce proper server-side authentication and authorization on every privileged page, regardless of the URL
- Treat security through obscurity as a minor hurdle, not a defense layer

**MITRE ATT&CK:** [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)