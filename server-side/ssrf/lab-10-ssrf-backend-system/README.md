# Lab: Basic SSRF against another back-end system

**Category:** SSRF
**Difficulty:** Apprentice
**Objective:** Scan the internal `192.168.0.X` range on port 8080 to find an admin interface, then delete user `carlos`

## 0. Attacker mindset

When an SSRF vulnerability exists, the server becomes a proxy into the internal network. Non routable private IP range (like 192.168.0.0/24) often host backend services that are not exposed to the internet. By using Intruder to iterate over the last octet, the attacker can map internal services and locate administrative interfaces that lack external protection

## 1. What is the vulnerability?

The stock check feature makes server-side requests to user-controllable URLs. Since the application does not restrict the target IP, an attacker can use the server to scan the internal network. In this case, an admin interface on port 8080 is accessible from the application server via the `192.168.0.X` range, but not from the outside

## 2. How did I exploit it?

1. Intercepted a `POST /product/stock` request in Burp Suite and sent it to Intruder
2. Set a Sniper attack on the last octet of the `stockApi` parameter: `http://192.168.0.§X§:8080/admin`
3. Configured payloads as numbers (1-254) and started the attack
4. Identified `192.168.0.89` and `192.168.0.1` with HTTP code 302 and 400, while all other IPs returned code 500. Finishing deletion of `carlos` and solving the lab.

## 3. Impact

- The attacker can use the application server as a pivot to scan and access internal services
- Administrative functionality on backend systems becomes reachable, bypassing perimeter firewalls and network segmentation
- Even if authentication mechanism exist on internal services, they may be weaker than those facing the internet

## 4. How can it be fixed?

- Do not allow user-supplied URLs in server-side requests; use a whitelist of allowed domains/IPs
- Block the server from making requests to internal IP ranges (RFC 1918) and loopback addresses
- Segment internal networks and enforce strict firewall policies between application servers and backend systems
- Apply the principle of least privilege: the application server should only have network access to services it strictly needs

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)