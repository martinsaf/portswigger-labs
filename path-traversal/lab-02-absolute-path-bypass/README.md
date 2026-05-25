# Lab: File path traversal, traversal sequences blocked with absolute path bypass

**Category:** Path Traversal\
**Difficulty:** Practitioner\
**Objective:** Retrieve the contents of the `/etc/passwd` file. 

## 0. Attacker mindset

The application blocks directory traversal sequences like `../`. However, it still allows user input to be used in a file path. If the application accepts absolute paths (starting with `/`), the base directory can be bypassed entirely. The attacker's goal is to supply an absolute path to a sensitive system file and see if the server reads it directly.

## 1. What is the vulnerability?

The `filename` parameter in the `/image` endpoint is concatenated to a base directory (`/var/www/images/`). The application strips or blocks `../` sequences, preventing directory traversal. However, it does not block paths that begin with `/` (absolute paths). When an absolute path is provided, some file system APIs ignore the base directory and use the absolute path directory.


## 2. How did I exploit it?

1. Identify a vulnerable endpoint: `/image?filename=36.jpg`
2. Attempted `filename=../../../etc/passwd` - the `../` sequences were blocked
3. Tried an absolute path instead: `GET /image?filename=/etc/passwd HTTP/2`
4. The server returned `HTTP/2 200 OK` with the full contents of `/etc/passwd` in the response body

## 3. Impact

- Direct read of arbitrary system files using absolute paths
- Bypass of basic `../` filtering without needing complex traversal sequences
- Exposure of sensitive operating system files (`/etc/passwd`, configuration files, application source code)

## 4. How can it be fixed?

- Do not accept user-supplied input for file paths. Use a whitelist of allowed filenames mapped to internal file paths
- If user input must be used, normalize the path and verify it stays within the intended base directory
- Reject any path that starts with `/` (absolute path) or cotains `..`
- Store files outside the web root and use a handler that does not expose direct file system access

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)