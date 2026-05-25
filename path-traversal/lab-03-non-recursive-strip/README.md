# Lab: File path traversal, traversal sequences stripped non-recursively

**Category:** Path Traversal\
**Difficulty:** Practitioner\
**Objective:** Retrieve the contents of the `/etc/passwd` file

## 0. Attacker mindset

The application strips `../` sequences from the filename before using it. However, if the stripping is **non-recursive** (only one pass), nested traversal sequences can bypass the defense. By placing `../` inside extra characters, the inner `../` is stripped, and the remaining characters reform into a valid `../` sequence.

## 1. What is the vulnerability?

The `filename` parameter is sanitized by removing `../` sequences. But the sanitization runs only once. An attacker can use nested traversal sequences like `....//`. When `../` is stripped from the middle, the remaining `../` becomes a valid directory traversal sequence.

**Example transformation:**
- Input: `....//etc/passwd`
- Strip `../` from the middle: `../` + `etc/passwd`

The working payload: `....//....//....//etc/passwd`

Each `....//` collapses into `../` after one stripping pass. Three of them get you from `/var/www/images/` to the root.

## 2. How did I exploit it?

1. Identified a vulnerable endpoint: `/image?filename=36.jpg`
2. Used nested traversal sequences: `GET /image?filename=....//....//....//etc/passwd HTTP/2`
3. The server returned `HTTP/2 200 OK` with the full contents of `/etc/passwd`

## 3. Impact

- Bypass of non-recursive path traversal stripping defenses
- Arbitrary file read on the server
- Exposure of sensitive system files (`/etc/passwd`, configuration, source code)

## 4. How can it be fixed?

- Use **recursive stripping** or normalization (remove `../` repeatedly until no more matches)
- Better yet, use a whitelist of allowed filenames instead of blacklisting sequences
- Normalize the path using `realpath()` and verify it starts with the intended base directory
- Do not rely on string replacement for security - use filesystem APIs that prevent path traversal by design

--- 

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)