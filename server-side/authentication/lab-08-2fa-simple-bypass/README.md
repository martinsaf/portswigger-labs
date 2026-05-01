# Lab: 2FA simple bypass

**Category:** Authentication
**Difficulty:** Apprentice
**Objective:** Bypass the two-factor authentication and access Carlos's account page.

## 0. Attacker mindset

Two-factor authentication is meant to add an extra layer of security, but if the application's state management is flawed, the second factor becomes optional. When the server grants a fully authenticated session immediately after the password step, the 2FA prompt is just a UI inconvenience rather than a real security check. An attacker who knows valid credentials can test whether protected pages are accessible before completing the second factor.

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

## 3. Impact

- Complete bypass of two-factor authentication
- Full access to a victim's account knowing only the password (which could have been obtained via brute-force, credential stuffing, or phishing)
- The 2FA mechanism provides no additional security, leaving accounts as vulnerable as if 2FA were not enabled
- Undermines trust in the authentication system for all users

## 4. How can it be fixed?

- The server must maintain the authentication state correctly: the session should not be marked as fully authenticated until **all** required factors are successfully completed
- Every request to a protected resource must check whether the current session has passed the complete authentication process, including the 2FA step 
- Do not grant access to any page that requires full authentication based solely on the password step being completed

**MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)
