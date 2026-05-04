# Lab: OS command injection, simple case

**Category:** OS Command Injection
**Difficulty:** Apprentice
**Objective:** Execute `whoami` via the product stock checker

## 0. Attacker mindset

When an application executes a shell command using my input, I can append command separators and inject additional shell commands. This is the path to Remote Code Execution (RCE) on the server

## 1. What is the vulnerability?

The stock check feature constructs a shell command (`stockreport.sh`) with the user-supplied `productId` and `storeId` as arguments. Since no sanitization is performed, shell metacharacters injected into these parameters allow arbitrary command execution

## 2. How did I exploit it?

1. Navigated to a product page and clicked "Check stock"
2. In DevTools -> **Network** -> Right-click the POST /stock request -> **Edit and Resend**
3. Tested injection with `productId=1 | echo hello123` - confirmed execution because the response contained "hello123" - message of lab solve poped up
4. For a clean output, injected `productId=3 & whoami #`, commenting out the remainder of the line
5. The response displayed the username `peter-4ScIXt`, proving command execution

## 3. Impact

- Remote Code Execution (RCE) on the web server
- Full system compromise, data theft, and lateral movement
- Ability to execute any command with the web server's privileges

## 4. How can it be fixed?

- Avoid calling OS commands directly. Use language APIs or libraries instead
- If unavoidable, strickly validate input against an allowlist (e.g., only numeric characters)
- Escape or sanitize shell metacharacters before passing to a command
- Run the application with least-privilege system permissions

**MITRE ATT&CK:** [T1059 - Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/)