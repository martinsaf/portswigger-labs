# Lab: File path traversal, traversal sequence stripped with superfluous URL-decode

**Category:** Path Traversal\
**Difficulty:** Practitioner\
**Objective:** Retrieve the contents of the `/etc/passwd` file

## 0. Attacker mindset

The application blocks input containing `../` sequences. However, it then performs a URL-decode on the input before using it. If the application does **double URL-decode** (superfluous decode), an attacker can use double-encoded traversal sequences. The blocking check sees the encoded version (which contains no `../`), then the double decode converts it to `../` after the check passes.

## 1. What is the vulnerability?

The application sanitizes user input by blocking any request containing `../`. Then it applices URL-decoding (possibly more than once) before using the input in a file path. This creates a race between the blocking check and the decode.

**Double URL-encode bypass:**
- `../` URL-encoded once: `%2e%2e%2f`
- `../` URL-encoded twice: `%252e%252e%252f`

The blocking check sees `%252e%252e%252f` - no `../` pattern (only `%25`). After two URL-decode passes:"
- Pass 1: `%252e%252e%252f` -> `%2e%2e%2f`
- Pass 2: `%2e%2e%2f` -> `../`

The ``../` appears **after** the blocking check, allowing path traversal.

## 2. How did I exploit it?

1. Identified a vulnerable endpoint: `/image?filename=26.jpg`
2. Constructed a double URL-encoded payload with three traversal sequences (to go from `/var/www/images/` to root): `GET /image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd HTTP/2`
3. The server returned `HTTP/2 200 OK` with the full contents of `/etc/passwd`

## 3. Impact

- Bypass of input filtering that blocks `../` sequences
- Arbitrary file read using double URL-encoding techniques
- Exposure of sensitive system files

## 4. How can it be fixed?

- Normalize and validate input **after** all decoding has been performed
- Apply blocking checks on the final decoded value, not on the raw input
- Use a whitelist of allowed filenames instead of blacklisting traversal sequences
- Use `realpath()` to resolve the final path and verify it starts with the intended base directory

---

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)