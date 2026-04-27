# Lab: User ID controlled by request parameter with password disclosure

**Category:** Access Control
**Difficulty:** Apprentice
**Objective:** Retrieve the administrator's password and use it to delete user `carlos`.

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

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:**
- [T1548 - Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)
- [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/) (using stolen administrator credentials)