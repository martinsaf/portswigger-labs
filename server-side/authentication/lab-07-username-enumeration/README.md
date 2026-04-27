# Lab: Username enumeration via different responses

**Category:** Authentication
**Difficulty:** Apprentice
**Objective:** Enumerate a valid username, brute-force the password, and access the account page.

## 1. What is the vulnerability?

The application responds differently depending on whether a submitted username exists in the database. When an invalid username is provided, the server returns a response with a consistent length. When a valid username is entered (even with a wrong password), the response length changes slightly. This behavior allows an attacker to build a list of valid usernames.

Once a valid username is identified, the same login endpoint can be brute-forced with a list of passwords. The application returns a different HTTP status code (302 redirect) when the correct password is supplied, making it trivial to identify the working credentials.


## 2. How did I exploit it?

1. Intercepted a login request (`POST /login`) with Burp Suite Proxy using dummy credentials `username:password`.
2. Sent the request to Burp Intruder with a **Sniper attack** on the `username` parameter: `username=§username§&password=password`
3. Loaded the candidate usernames wordlist into the payloads and started the attack.
4. Analyzed the response lengths. All invalid usernames returned the same length, except for one — `an` — which had two extra bytes.
5. Modified the Intruder attack to target the `password` parameter with the known username: `username=an&password=§password§`
6. Cleared the previous payloads and loaded the candidate passwords wordlist.
7. Started the attack and identified a single response with status code 302 (redirect). The corresponding password was `monitoring`.
8. Logged in with `an:monitoring`. The lab solved automatically upon successful authentication.

## 3. How would I detect this in my home lab?
*To be tested*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)