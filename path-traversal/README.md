# Path traversal (Directory Traversal)

Path traversal vulnerabilities allow an attacker to read (and sometimes write) arbitrary files on the server. This can lead to disclosure of source code, credentials, sensitive system files, and ultimately take full control of the server.

## Common impact

- Unauthorized read access to configuration files, credentials, and logs
- Exposure of application source code and intellectual property
- In write scenarios, modification of application behavior and code execution

## Labs

1. [File path traversal, simple case](/server-side/path-traversal/lab-01-simple/README.md) - Apprentice
2. [File path traversal, traversal sequences blocked with absolute path bypass](./lab-02-absolute-path-bypass/README.md) - Practitioner
3. [File path traversal, traversal sequences stripped non-recursively](./lab-03-non-recursive-strip/README.md) - Practitioner

## Reference

[OWASP Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)