# PortSwigger Web Security Labs

Documenting my progress through the [PortSwigger Web Security Academy](https://portswigger.net/web-security).

## What this is

A collection of write-ups for each lab I solve. Each one documents the vulnerability, the exploitation step, and the detection hypothesis - even when I don't yet have the infrastructure to test it fully.

## Structure

```
D:\Dev\GitHub\martinsaf\portswigger-labs>tree /f
Folder PATH listing for volume EXTERNAL_USB
Volume serial number is 7879-3971
D:.
│   .gitattributes
│   LICENSE
│   README.md
│
├───server-side
│   │   README.md
│   │
│   ├───access-control
│   │   │   README.md
│   │   │
│   │   ├───lab-02-unprotected-admin
│   │   │       README.md
│   │   │
│   │   ├───lab-03-unpredictable-url
│   │   │       README.md
│   │   │
│   │   ├───lab-04-role-cookie
│   │   │       README.md
│   │   │
│   │   ├───lab-05-horizontal-guid
│   │   │       README.md
│   │   │
│   │   └───lab-06-horizontal-to-vertical
│   │           README.md
│   │
│   ├───authentication
│   │   │   README.md
│   │   │
│   │   ├───lab-07-username-enumeration
│   │   │       README.md
│   │   │
│   │   └───lab-08-2fa-simple-bypass
│   │           README.md
│   │
│   ├───file-upload
│   │   │   README.md
│   │   │
│   │   ├───lab-11-web-shell-upload
│   │   │       exploit_script.php
│   │   │       README.md
│   │   │       versatile_exploit_script.php
│   │   │
│   │   └───lab-12-content-type-bypass
│   │           README.md
│   │
│   ├───os-command-injection
│   │   │   README.md
│   │   │
│   │   └───lab-13-simples-case
│   │           README.md
│   │
│   ├───path-traversal
│   │   │   README.md
│   │   │
│   │   └───lab-01-simple
│   │       │   README.md
│   │       │
│   │       └───screenshots
│   │               lab-01-passwd.txt
│   │               lab-01-response.jpg
│   │
│   ├───sql-injection
│   │   │   README.md
│   │   │
│   │   ├───lab-14-retrieve-hidden-data
│   │   │       README.md
│   │   │
│   │   └───lab-15-login-bypass
│   │           README.md
│   │
│   └───ssrf
│       │   README.md
│       │
│       ├───lab-09-basic-ssrf
│       │       README.md
│       │
│       └───lab-10-ssrf-backend-system
│               README.md
│
└───sql-injection
    │   README.md
    │
    ├───lab-01-union-columns
    │       README.md
    │
    └───lab-02-find-text-column
            burp-notes.md
            README.md
```

## Why 

I'm building a home SOC lab (Windows AD, Linux, Wazuh SIEM). This repository is where I study the offensive side - web application vulnerabilities - to better understand what to look for on the defensive side.

Currently, I don't have a public web server in my lab, so the detection sections are hypotheses. When that changes, I'll revisit and update them.

## Status

*In progress.*

Paths:
- Server-side vulnerabilities: 52 of 52 Completed
- SQL injection: 16 of 51 Completed
