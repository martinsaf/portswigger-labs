# Lab: Basic SSRF against the local server

**Category:** SSRF
**Difficulty:** Apprentice
**Objective:** Access the admin interface at `http://localhost/admin` and delete user `carlos`

## 0. Attacker mindset

The application allows user-controlled input to define a backend request.
This immediately suggests a potential SSRF vulnerability, as the server may be used as a proxy to access internal resources.

Given that internal services often trust localhost, the goal is to pivot the request to 127.0.0.1 and access internal endpoints such as administrative interfaces.

## 1. What is the vulnerability?

The application fetches stock information by making a server-side request to a URL supplied by the user in the `stockApi` parameter. Because the application does not validate or restrict the target of this request, an attacker can provide a URL pointing to internal services, including the server's own loopback address (`localhost` or `127.0.0.1`)

The administrative interface at `/admin` is normally inaccessible from the outside, but requests originating from the local machine bypass access controls. By forcing the server to request `http://localhost/admin`, the attacker can access the admin panel and perform privileged actions, such as deleting users

## 2. How did I exploit it?

1. Navigated to a product page and clicked "Check stock" while Burp Suite was intercepting traffic
2. Identified the `POST /product/stock` request containing the `stockApi` parameter: `stockApi=http%3a%2f%2fstock.weliketoshop.net%3a8080%2fproduct%2fstock%2fcheck%3fproductId%3d2%26storeId%3d1`
3. Sent the request to Burp Repeater and changed the URL-decoded `stockApi` parameter to `http://localhost/admin`
4. Sent the request and received the admin interface HTML in the response. Observed the link for deleting users: `/admin/delete?username=carlos`
5. Modified the `stockApi` parameter to `http://localhost/admin/delete?username=carlos` and sent the request again. The user `carlos` was deleted, lab solved.

## 3. Impact

This vulnerability allows an attacker to access internal services that are not exposed externally.

In this lab, it leads to:
- Unauthorized access to the admin panel
- Ability to perform privileged actions (e.g., deleting users)

In real-world scenarios, SSRF can also be used to:
- Access internal APIs
- Retrieve sensitive data
- Interact with other systems inside the network

## 4. How can it be fixed?

- Do not allow user-controlled URLs to be used directly in server-side requests
- Validate and restrict input to only trusted destinations (allowlist)
- Block requests to internal IP addresses such as 127.0.0.1 and private network ranges
- Avoid making unnecessary outbound requests from the server


**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
