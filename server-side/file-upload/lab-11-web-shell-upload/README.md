#  Lab: Remote code execution via web shell upload

**Category:** File Upload
**Difficulty:** Apprentice
**Objective:** Upload a PHP web shell, then exfiltrate the contents of `/home/carlos/secret`

## 0. Attacker mindset

An unrestricted file upload is a direct path to RCE on the server. If a website allows me to upload executable files (PHP, JSP, etc.) and then gives me a direct URL to them, I can run arbitrary code on their machine - reading files, executing commands, or even setting up a reverse shell.

## 1. What is the vulnerability?

The application's avatar upload function does not perform any validation on the type or content of uploaded files. The user can upload a PHP script instead of an image. The server saves the file and makes it accessible via a public URL. When that URL is visited, the server executes the PHP code, allowing the attacker to run commands or read files.

## 2. How did I exploit it?

1. Logged in as `wiener:peter`
2. Navigated to the "My account" page and used the avatar upload functionality.
3. Intercepted the upload request with Burp Suite and replaced the image file content with a PHP script: [exploit script](./exploit_script.php)
4. Forwarded the request. The server saved the file as my new avatar
5. On the "My account" page, right-clicked the avatar image icon and selected "Open image in new tab" to determine the direct URL of the uploaded file
6. The PHP script executed and displayed the secret string in the browser: `zUnWmMuRVAumKumc6f1DifPyhu94Jrhx`
7. Submitted the secret to solve the lab.

## 3. Impact

- Complete confidentiality failure: arbitrary files on the server can be read (e.g., `home/carlos/secret` configuration files, source code)
- Potencial for full remote code execution and system takeover if a more capable shell is uploaded (e.g., `system($_GET['cmd'])`)
- Server can be used as a pivot point for further internal network attacks

## 4. How can it be fixed?

- **Never trust user input** Validate all uploaded files:
    * Check the file extension against a whitelist (e.g., `.jpg`, `.png`)
    * Verify the MIME type in the request and perform server-side content inspection
    * Store uploaded files outside the web root or with a randomized name to prevent direct execution
- Configure the web server to not execute scripts in upload directories
- Set appropriate filesystem permissions on upload folders to limit the impact of any malicious file that does sneak through

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)