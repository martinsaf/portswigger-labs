# Server-side Vulnerabilities

This module covers my documentation of the PortSwigger Web security Academy's **Server-side vulnerabilities** path (labs 1-52).

The goal is not just to solve labs, but to understand the root cause of each vulnerability, how its exploitation manifests in system and network logs, and how I would detect and prevent it in my home SOC lab.

## Categories

- [Path Traversal](./path-traversal/) (3 labs)
- Access Control (coming next)
- Authentication
- Server-Side Request Forgery (SSRF)
- File Upload Vulnerabilities
- OS Command Injection
- SQL Injection

## Approach

For every lab completed:

1. **Exploit** the vulnerability.
2. **Document** the exact steps.
3. **Design** a detection rule (Sigma, Wazuh, Sysmon/Auditd) based on observable artifacts.
4. **Map** to MITRE ATT&CK

All write-ups include a defensive perspective, connecting the attack to logs I would monitor in my home lab (Apache, Windows Event Logs, Sysmon, Auditd, Wazuh).

## Home Lab Context

My home lab runs Windows AD, Linux endpoints, Apache web servers, and Wazuh SIEM. This documentation extends that lab by simulating offensive techniques and designing detection logic for them.