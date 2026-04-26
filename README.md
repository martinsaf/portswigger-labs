# PortSwigger Web Security Labs

Documenting my progress through the [PortSwigger Web Security Academy](https://portswigger.net/web-security).

## What this is

A collection of write-ups for each lab I solve. Each one documents the vulnerability, the exploitation step, and the detection hypothesis - even when I don't yet have the infrastructure to test it fully.

## Structure

```
portswigger-labs/
│   README.md
│
└───server-side
    │   README.md
    │
    ├───access-control
    │   │   README.md
    │   │
    │   ├───lab-02-unprotected-admin
    │   │       README.md
    │   │
    │   ├───lab-03-unpredictable-url
    │   │       README.md
    │   │
    │   ├───lab-04-role-cookie
    │   │       README.md
    │   │
    │   └───lab-05-horizontal-guid
    │           README.md
    │
    └───path-traversal
        │   README.md
        │
        └───lab-01-simple
            │   README.md
            │
            ├───detection
            └───screenshots
                    lab-01-passwd.txt
                    lab-01-response.jpg
```

## Why 

I'm building a home SOC lab (Windows AD, Linux, Wazuh SIEM). This repository is where I study the offensive side - web application vulnerabilities - to better understand what to look for on the defensive side.

Currently, I don't have a public web server in my lab, so the detection sections are hypotheses. When that changes, I'll revisit and update them.

## Status

In progress. Labs solved: 5/52