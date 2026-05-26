# Path traversal (Directory Traversal)

Path traversal vulnerabilities allow an attacker to read (and sometimes write) arbitrary files on the server. This can lead to disclosure of source code, credentials, sensitive system files, and ultimately take full control of the server.

## Common impact

- Unauthorized read access to configuration files, credentials, and logs
- Exposure of application source code and intellectual property
- In write scenarios, modification of application behavior and code execution

## Labs

| Lab | Técnica principal |
| - | - |
| 1. [File path traversal, simple case](/server-side/path-traversal/lab-01-simple/README.md) - Apprentice | `../../../etc/passwd` básico |
| 2. [File path traversal, traversal sequences blocked with absolute path bypass](./lab-02-absolute-path-bypass/README.md) - Practitioner | `/etc/passwd` (absolute path) | 
| 3. [File path traversal, traversal sequences stripped non-recursively](./lab-03-non-recursive-strip/README.md) - Practitioner | `....//....//....//etc/passwd` |
| 4. [File path traversal, traversal sequences stripped with superfluous URL-decode](./lab-04-superfluous-url-decode/README.md) - Practitioner | `%252e%252e%252f` (double URL encode) |
| 5. [File path traversal, validation of start of path](./lab-05-validation-of-start-path/README.md) - Practitioner | `/var/www/images/../../../etc/passwd` | 
| 6. [File path traversal, validation of file extension with null byte bypass](./lab-06-null-byte-extension-bypass/README.md) - Practitioner | `../../../etc/passwd%00.png` |

## How to prevent path traversal attacks

The most effective way is to **avoid passing user-supplied input to filesystem APIs altogether**. If that's not possible, use two layers of defense:

1. **Validate user input** - Use a whitelist of permitted values. If not possible, restrict to alphanumeric characters only
2. **Canonicalize the path** - Append input to the base directory, then use a file system API to get the canonical (absolute) path. Verify that the canonicalized path **starts with** the expected base directory

**Example (Java):**
```java
File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // process file
}
```

## Reference

[OWASP Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)