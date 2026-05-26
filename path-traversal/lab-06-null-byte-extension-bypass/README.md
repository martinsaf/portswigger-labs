# Lab: File path traversal, validation of file extension with null byte bypass

**Category:** Path Traversal\
**Difficulty:** Practitioner\
**Objective:** Retrieve the contents of the `/etc/passwd` file.

## 0. Attacker mindset

The application validates that the filename ends witha a specific file extension (e.g., `.png`). If the validation only checks the string suffix but the underlying filesystem API treats null bytes as string terminators, an attacker can inject `%00` (URL-encoded null byte) to truncate the string. The validation sees the expected extension, but the filesystem sees only the path before the null byte.

## 1. What is the vulnerability?

The application requires the `filename` parameter to end with an extension like `.png`. However the filesystem API (common in older PHP versions, < 5.3.4) stops reading the string when it encounters a null byte (`\0`). By URL-encoding the null byte as `%00`, the attacker can:

- Satisfy the extension validation: the string ends with `.png`
- Truncate the path before the extension: everything after `%00` is ignored

**Example:**
- Input: `../../../etc/passwd%00.png`
- Validation sees: `../../../etc/passwd%00.png` (ends with `.png`? -> passes)
- Filesystem sees: `../../../etc/passwd` (null byte terminates the string)
- Resolved path: `/etc/passwd`

## 2. How did I exploit it?

1. Identified a vulnerable endpoint: `/image?filename=36.jpg`
2. Constructed a payload with path traversal sequences followed by a null byte and a valid extension: `GET /image?filename=../../../etc/passwd%00.png HTTP/2`
3. The server returned `HTTP/2 200 OK` with the full contents of `/etc/passwd`
4. Note: The `Content-Type` in the response was `image/png` (the server thought it was serving an image), but the odfy contained the passwd file.

## 3. Impact

- Bypass of file extension validation using null byte injection
- Arbitrary file read on the server
- Exposure of sensitive system files (`/etc/passwd`, configuration, source code)

## 4. How can it be fixed?

- Use modern, secure filesystem APIs that handle null bytes as regular characters
- Do not rely on string suffix validation alone - validate the file type using content inspection (magic byte)
- Use a whitelist of allowed filenames mapped to internal identifiers
- Reject any user input containing null bytes (`%00`, `\0`) before processing

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)