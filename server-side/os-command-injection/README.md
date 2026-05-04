# OS Command Inkection

[OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)

OS command injection allows an attacker to execute arbitrary commands on the server's operating system by injecting shell metacharacters into user-supplied input that is passed to a shell command.

## Common impact
- Remote code execution (RCE) on the underlying OS
- Full server server compromise, data exfiltration, lateral movement
- Execution of system commands with the privileges of the web server process

## Labs
1. [Simple case](./lab-13-simple-case/) - Apprentice