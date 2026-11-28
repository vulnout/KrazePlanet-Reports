## Title
Stored XSS - Video Streaming Service (Netflix Clone)
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://netflix.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Feature:** Sign Up Modal (`Username` field)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Username** field of the user registration (**Sign Up**) form on `https://netflix.pwnedbox.site`. User-supplied input submitted during account creation is stored in the backend database and rendered unescaped across the application (such as in the user profile header, movie reviews, or user listings) without proper HTML entity encoding or sanitization.

Because arbitrary HTML tags and event handlers (such as `<img onError=...>`) are executed directly in the browser, any user or administrator who views content associated with the attacker's account—or the attacker's own active session—will have malicious JavaScript executed automatically.
## Steps to Reproduce
1. Navigate to `https://netflix.pwnedbox.site` and click **Sign In** -> **Sign Up**.
2. Fill in **Full Name**, **Email Address**, and **Password** with valid details, and enter `'">abmi5<img src=O onError=alert(5)>` into the **Username** field.
3. Click **Create Account & Sign Up**.
4. Observe that `alert(5)` executes automatically when the stored username is rendered on the page.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. XSS payload entered into the `Username` field during registration on `https://pwnedbox.site`**

<img width="1000" height="512" alt="image" src="https://github.com/user-attachments/assets/11626e55-0302-4842-9062-9e149998d637" />

**2. Arbitrary JavaScript execution (`alert(5)`) triggered automatically upon rendering the stored username**

<img width="1600" height="797" alt="image" src="https://github.com/user-attachments/assets/6e56778e-1534-4056-9756-39ac12edf8a1" />

## Impact
Because usernames are persistently stored and displayed publicly whenever a user posts a review, interacts on the platform, or is viewed in an admin panel, this Stored XSS vulnerability affects any authenticated user or administrator who encounters the attacker's username. Successful exploitation allows an attacker to:
* Hijack user or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as modifying account credentials, posting fraudulent reviews, or escalating privileges) on behalf of victims.
* Redirect visitors to malicious phishing pages under the trusted `netflix.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Username` and `Full Name` fields using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them anywhere in the UI.
* **Strict Input Validation:** Enforce an allowlist regex (e.g., `^[a-zA-Z0-9_.-]{3,20}$`) on the `Username` field on the server side to reject HTML special characters (`<`, `>`, `"`, `'`) at registration.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
