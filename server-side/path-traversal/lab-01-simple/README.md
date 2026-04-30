# Lab: File path traversal, simple case

**Category:** Path Traversal
**Difficulty:** Apprentice
**Objective:** Retrieve the contents of `/etc/passwd`.

## 0. Attacker mindset

Any user-supplied input that is used to construct a file path is a candidate for path transversal. If the application prepends a base directory but does not sanitize the input, injecting `../` sequences can escape the intended folder and reach arbitrary files on the system.

The presence of a `filename` parameter in an image loading endpoint is a strong signal to test traversal payloads.

## 1. What is the vulnerability?

The application uses the `filename` parameter directly to build a file path on the server by concatenating it to the base directory `/var/www/images`. For example, `filename=218.png` returns the file `/var/www/images/218.png`.

The problem is that the application does not validate or sanitize the input. This allows an attacker to inject the `../` sequence, which moves up one level in the directory tree. By using `../../../etc/passwd`, the server ends up reading the `/etc/passwd` file at the system root. The flaw exists on both Unix (`../`) and Windows (`..\`) systems, and exposes any file the web server process has read access to.

## 2. How did I exploit it?

1. Identified the vulnerable endpoint: `image?filename=41.jpg` (any image will work)
2. Attempted path traversal with incremental ../ sequences:
    - /etc/passwd -> "no such file" (path still relative to images dir)
    - ../etc/passwd -> "no such file"
    - ../../etc/passwd -> "no such file"
    - ../../../etc/passwd -> 200 OK (file found, served as image/jpeg)
3. Viewed the response body via DevTools Network tab -> Response to confirm file contexts.
4. Extracted /etc/passwd containing system user accounts.

![/etc/passwd screenshoot extraction](./screenshots/lab-01-response.jpg)
*/etc/passwd screenshoot extraction*

[Content of /etc/passwd extracted](screenshots/lab-01-passwd.txt)

## 3. Impact

A successful path traversal can lead to:
- Disclosure of sensitive operating system files (e.g., `/etc/passwd`, `/etc/shadow`, configuration files)
- Leakage of application source code and hardcoded credentials
- In servere cases, writing to arbitrary files, leading to remote code execution

## 4. How can it be fixed?

- Never concatenate user input directly into file paths
- Use a whitelist of allowed filenames and map them to actual files on the server
- If user input must be part of the path, normalize the path using functions like `realpath()` and verify that it starts with the expected base directory
- Run the web server process with minimal file system permissions

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

