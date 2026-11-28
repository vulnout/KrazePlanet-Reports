## Title
Stored XSS - Twitter
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://ads.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Feature:** New Network Report Modal (`Report Name` field)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Report Name** field of the **New Network Report** feature on AdPulse (`https://ads.pwnedbox.site`). User-supplied input entered when creating a network report is stored in the backend database and rendered unescaped inside the shared **Network Reports** dashboard table without proper HTML entity encoding or sanitization.

As noted on the dashboard (*"All network reports — visible to every authenticated user"*), any authenticated user or administrator who views the **Network Reports** page will automatically execute the attacker's arbitrary JavaScript in their browser session.

## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1.Create account**

<img width="1600" height="802" alt="image" src="https://github.com/user-attachments/assets/aaecac53-d00c-4473-b313-f7a223b88375" />

**2. XSS payload entered into the `Report Name` field on `https://ads.pwnedbox.site`**

<img width="1600" height="793" alt="image" src="https://github.com/user-attachments/assets/a1905b4b-f39e-4076-ae91-5cf71f2a2352" />

**3. Observe Javascript executed**

<img width="1600" height="802" alt="image" src="https://github.com/user-attachments/assets/09950e98-471f-4f40-ab7d-ea7f3380dec2" />

## Impact
Because network reports are shared globally across the platform and visible to every authenticated user and administrator, this Stored XSS vulnerability compromises all users who visit the **Network Reports** dashboard. Successful exploitation allows an attacker to:
* Hijack user or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as modifying or deleting ad reports, altering account settings, or escalating privileges) on behalf of victims.
* Redirect platform users to malicious phishing pages under the trusted `ads.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Report Name` field using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering it inside the Network Reports table.
* **Server-Side Input Validation:** Restrict report names to expected alphanumeric and punctuation characters on the server side to reject HTML tags and event handlers.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
