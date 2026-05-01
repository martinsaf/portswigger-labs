# Path Traversal (Directory Traversal)

[OWASP Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)

Path traversal vulnerabilities allow an attacker to read (and sometimes write) arbitrary files on the server. This can lead to disclosure of source code, credentials, sensitive system files, and ultimately take full control of the server.

## Common impact

- Unauthorized read access to configuration files, credentails, and logs
- Exposure of application source code and intellectual property
- In write scenarios, modification of application behaviour and code execution

## Lab

1. [File path, simple case](./lab-01-simple/) - Apprentice