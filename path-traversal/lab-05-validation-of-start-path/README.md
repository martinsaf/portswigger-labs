# Lab: File path traversal, validation of start of path

**Category:** Path Traversal\
**Difficulty:** Practitioner\
**Objective:** Retrieve the contents of the `/etc/passwd` file.

## 0. Attacker mindset

The application validates that the supplied filename starts with a specific base path (e.g., `/var/www/images/`). If the path does not start with the expected prefix, the request is rejected. However, the validation only checks the **beginning** of the string, not the final resolved path. By including the required prefix followed by `../` sequences, the attacker can step out of the base directory and reach any file on the system.

## 1. What is the vulnerability?

The application expects the `filename` parameter to contain a full path that begins with a trusted base directory (e.g., `/var/www/images/`). The validation verifies this prefix but does not restrict what comes after. When the path is used, the operating system resolves the `../` sequences, effectively ignoring the prefix.

**Example:**
- Input: `/var/www/images/../../../etc/passwd`
- Validation: starts with `/var/www/images` -> passes
- Resolved path: `/etc/passwd`
- Result: the contents of `/etc/passwd` are returned

## 2. How did I exploit it?

1. Identified the vulnerable endpoint: `/image?filename=36.jpg`
2. Observed that images are served from `/var/www/images` (by inspecting an existing image request or checking the HTML source)
3. Constructed a payload that includes the required prefix followed by traversal sequences: `GET /image?filename=/var/www/images/../../../etc/passwd HTTP/2`
4. The server returned `HTTP/2 200 OK` with the full contents of `/etc/passwd`

## 3. Impact

- Bypass of path prefix validation using `../` sequences
- Arbitrary file read on the server
- Exposure of sensitive system files (`/etc/passwd`, configuration, source code)

## 4. How can it be fixed?

- Use `realpath()` or equivalent to resolve the full filesystem path, then verify it still starts with the intended base directory
- Do not rely on string prefix validation alone - normalize the path first
- Use a whitelist of allowed filenames instead of constructing paths from user input
- Store files outside the web root and serve them via a handler that maps identifiers to internal file paths

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)