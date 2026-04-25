# Path Traversal (Directory Traversal)

Path traversal vulnerabilities allow an attacker to read (and sometimes write) arbitrary files on the server. This can lead to disclosure of source code, credentials, sensitive system files, and ultimately take full control of the server.

## Labs

1. [File path, simple case](./lab-01-simple/) - Apprentice
2. [File path traversal, traversal sequences blocked with absolute path bypass](./lab-02-absolute-path-bypass/) - Practitioner *(a preencher quando chegar lá)*
3. [File path traversal, traversal sequences stripped non-recursively](./lab-03-stripped-non-recursively/) - Practitioner *(a preencher quando chegares)*

## Detection Strategy (Home Lab)

- **Data sources:** Apache access logs, Wazuh agent.
- **Key indicators:** Requests containing `../`, `..\`, URL-encoded sequences, or attempts to access files like `/etc/passwd`, `/etc/shadow`, `win.ini`.
- **Sigma rule:** [path_traversal_web.yml](../detection-rules/path_traversal_web.yml)(link conceptual para regras que irei criando).

When multiple labs are done, this space will also compare detection effectiveness across different traversal techniques.