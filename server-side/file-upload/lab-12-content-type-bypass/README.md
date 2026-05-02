# Lab: Web shell upload via Content-Type restriction bypass

**Category:** File Upload
**Difficulty:** Apprentice
**Objective:** Upload a PHP web shell and exfiltrate `/home/carlos/secret`

## 0. Attacker mindset

A web application that validates file uploads solely by checking the `Content-Type` header is relying on client-supplied metadata. This header is trivially modifiable with an intercepting proxy. If the server does not inspect the actual file contents, an attacker can bypass the restriction by simply changing the declared MIME type to one that the server accepts (e.g., `image/jpeg`)

## 1. What is the vulnerability?

The image upload function attempts to enforce a file type restriction by checking the `Content-Type` header in the multipart request body. It does not perform any server-side inspection of the file's content or magic bytes. Because the `Content-Type` value is entirely controlled by the client, the validation can be circumvented by modifying the header in a proxy before the request reaches the server.

## 2. How did I exploit it?

1. Logged in as `wiener:peter` and navigated to the avatar upload function. 
2. Prepared a PHP script to read the secret file: [exploit script](/server-side/file-upload/lab-11-web-shell-upload/exploit_script.php)
3. Attempted to upload the `.php` file. The server rejected it based on the file type
4. Intercepted the rejected upload request with Burp Suite Proxy and sent it to Repeater
5. In the request body, located the `Content-Type` header associated with the file part and changed its value from `application/octet-stream` to `image/jpeg`
6. Sent the modified request. The server accepted the upload and returned: `The file avatars/exploit_script.php has been uploaded.`
7. Opened the uploaded file URL in the browser. The PHP script executed, displaying the secret
8. Submitted the secret to solve the lab

## 3. Impact

- Full bypass of file type validation, enabling upload of executable scripts
- Remote code execution (RCE) leading to arbitrary file disclosure
- Complete compromise of the server, data theft, and potential pivoting into internal networks

## 4. How can it be fixed?

- Never rely solely on client-supplied `Cotent-Type` headers for file validation
- Validate file type using server-side checks such as magic byte inspection or dedicated libraries (e.g., `libmagic`)
- Use a whitelist of allowed file extensions and verify file signatures independently of MIME headers
- Store uploaded files outside the web root or configure the server to not execute script in upload directories

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)