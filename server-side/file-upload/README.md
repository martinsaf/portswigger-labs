# File Upload Vulnerabilities

[OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

File upload vulnerabilities allow atackers to place malicious files on the server's filesystem by abusing missing or flawed validation of uploaded content. In the worst case, this case leads to remote code execution (RCE).

## Common impact

- Upload of web shells granting full server control (RCE)
- Defacement, data theft, malware distribution
- Pivoting into internal networks from the compromised server

## Labs

1. [Remote code execution via web shell upload](./lab-11-web-shell-upload/README.md) - Apprentice

