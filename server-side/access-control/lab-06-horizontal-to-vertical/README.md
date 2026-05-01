# Lab: User ID controlled by request parameter with password disclosure

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Retrieve the administrator's password and use it to delete user `carlos`.

## 0. Attacker mindset

An IDOR that exposes another user's account page is a horizontal escalation opportunity. But if that page leaks credentials - such as a password pre-filled in a masked input - the attacker can escalate vertically by stealing an admin's password. The approach is to enumerat user identifiers, access privileged accounts, and extract any sensitive data that appears in the page source.

## 1. What is the vulnerability?

The application uses a user-supplied `id` parameter to determine which account page to display, but it does not verify whether the logged-in user is authorized to access that account. This is an Insecure Direct Object Reference (IDOR) that allows horizontal privilege escalation.

Even worse, the account page shows the current user's password pre-filled in a masked input field. The password is obfuscated in the UI but appears in clear text in the HTML source. An attacker can simply change the `id` parameter to an administrator's username, view the page source, and read the password. With stolen admin credentials, the attacker can then escalate vertically and perform priviledged actions.

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Noticed that the account URL contained `id=wiener`: `/my-account?id=wiener`
3. Changed the `id` parameter to `administrator` and loaded the page. The application returned the administrator's account page without any authorization check
4. Viewed the page source and found a password input with the administrator's password in the `value` attribute:
`<input required type=password name=password value='ibdy1d4czlco5gaufyqn'/>`
5. Logged out and logged in as `administrator` using the stolen password `ibdy1d4czlco5gaufyqn`
6. Accessed the admin panel and deleted user `carlos` to solve the lab.

## 3. Impact

- Horizontal escalation to an administrator's account via IDOR
- Exposure of the admin's password in client-side HTML, enabling full account takeover
- Complete compromise of administrative functionality (user deletion, data manipulation)
- Combination of horizontal and vertical privilege escalation in a single attack flow

## 4. How can it be fixed?

- Enforce strict server-side authorization checks: verify that the logged-in user owns the account identified by the `id` parameter before returning any data
- Never include the current password (or any sensitive credentials) in the HTML source; use a secure password-change flow that requires the user to supply the current password without it being served by the server
- Do not expose user identifiers that allow horizontal enumeration; use session-based references instead of direct object references in URLs

**MITRE ATT&CK:**
- [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)
- [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/) (using stolen administrator credentials)