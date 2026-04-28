# Lab: 2FA simple bypass

**Category:** Authentication
**Difficulty:** Apprentice
**Objective:** Bypass the two-factor authentication and access Carlos's account page.

## 1. What is the vulnerability?

The application implements a two-step authentication flow: first the user enters their password, then a 2FA verification code. However, the server considers the user fully authenticated after the password step, before the 2FA code is validated. This means the session is granted full access immediately upon password verification.

Because the server does not enforce the 2FA check on every subsequent request, an attacker who knows the victim's password can skip the verification code prompt entirely. By navigating directly to a protected page (e.g., `/my-account`) after the password step, the attacker bypasses the second factor and gains access to the victim's account.

## 2. How did I exploit it?

1. Logged in as `wiener:peter`. The application prompted for a 2FA verification code, which was sent by email. Accessed the email client, retrieved the code, and completed the 2FA step
2. Once logged in, noted the URL of the account page: `/my-account`
3. Logged out
4. Logged in as `carlos:montoya`. The application prompted for the 2FA verification code
5. Instead of entering the code, manually changed the URL to `/my-account`
6. The account page for `carlos` loaded immediately, bypassing the 2FA requirement. The lab was solved

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)
