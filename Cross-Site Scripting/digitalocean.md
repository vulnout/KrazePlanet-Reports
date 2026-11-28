## Title
Stored XSS - Support Center (DigitalOcean Clone)
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://digitalocean.pwnedbox.site/

**Affected Feature:** Create Support Ticket (`Subject` and `Description` fields)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in both the **Subject** and **Description** fields of the **Create Support Ticket** feature (`https://digitalocean.pwnedbox.site`). User-supplied input submitted when opening a support case is stored in the backend database and rendered unescaped on the **Support Tickets** dashboard without proper HTML entity encoding or sanitization.
Because support tickets are reviewed by support operators and administrators, any staff member or administrator who views the **Support Tickets** queue will automatically execute the attacker's arbitrary JavaScript in their privileged browser session.

## Steps to Reproduce
1. Register an account at `https://digitalocean.pwnedbox.site` and log in.
2. Navigate to **Create Support Ticket** ).
3. Enter `'">abmi5<img src=O onError=alert(1)>` into **Subject**, select any **Priority**, and enter `'">abmi5<img src=O onError=alert(2)>` into **Description**.
4. Click **Submit Support Case** and observe that upon redirecting, both stored payloads execute automatically—triggering `alert(1)` and `alert(2)`.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Register new account**

<img width="1599" height="804" alt="image" src="https://github.com/user-attachments/assets/bbcb3748-f87f-4ec3-9937-d559599e6bcb" />

**2. Log in into account**

<img width="1600" height="775" alt="image" src="https://github.com/user-attachments/assets/3f5783b9-3c0b-4662-aed1-f0e3c75da8d4" />

**3. XSS payloads entered into the `Subject` and `Description` fields**

<img width="1599" height="799" alt="image" src="https://github.com/user-attachments/assets/1a9cac7a-db38-41cb-8859-d398b246d583" />

**4. Observe Javascript execution**

<img width="1596" height="783" alt="image" src="https://github.com/user-attachments/assets/a0061e71-5d64-4ad7-b3e9-728cfae3b623" />

<img width="1600" height="791" alt="image" src="https://github.com/user-attachments/assets/af1b9409-f52d-4340-8769-2fe97535944c" />


## Impact
Because support tickets are directly viewed by support staff and portal administrators, this Stored XSS vulnerability enables **privilege escalation / administrative account takeover**. When an administrator views the ticket list or ticket details, the injected JavaScript executes automatically in their session, allowing an attacker to:
* Hijack administrator or support operator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized administrative actions (such as modifying user accounts, viewing confidential tickets, or escalating privileges) on behalf of the administrator.
* Redirect support personnel or users to malicious phishing pages under the trusted `digitalocean.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Subject` and `Description` fields using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them in the Support Tickets dashboard.
* **Server-Side Sanitization:** If basic formatting is needed in ticket descriptions, sanitize input on the server side using a strict allowlist-based sanitizer (such as **DOMPurify** or **HTML Purifier**) to strip dangerous tags and event handlers (`onError`, `onLoad`, etc.).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
