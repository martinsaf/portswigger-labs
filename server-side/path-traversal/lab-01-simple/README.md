# Lab: File path traversal, simple case

**Category:** Path Traversal
**Difficulty:** Apprentice
**Objective:** Retrieve the contents of `/etc/passwd`.

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

## 3. How would I detect this in my home lab?
*To be tested.*

## 4. How can it be fixed?
*To be tested*

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application]

