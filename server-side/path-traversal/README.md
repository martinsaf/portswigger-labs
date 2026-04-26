# Path Traversal (Directory Traversal)

Path traversal vulnerabilities allow an attacker to read (and sometimes write) arbitrary files on the server. This can lead to disclosure of source code, credentials, sensitive system files, and ultimately take full control of the server.

## Lab

1. [File path, simple case](./lab-01-simple/) - Apprentice

## Detection Strategy

**Hypotheses (to be tested when a web server is added to my lab):**

- **Data sources:** Apache/Nginx access logs.
- **Key indicators:** Requests containing `../`, `..\`, URL-encoded traversal
  sequences (`%2e%2e%2f`), or attempts to access system files like
  `/etc/passwd` or `win.ini`.
- **Rule idea:** A Sigma rule matching these patterns in web server logs,
  mapped to MITRE ATT&CK T1190.