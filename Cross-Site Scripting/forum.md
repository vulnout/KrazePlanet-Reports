## Title
Stored XSS - Acronis
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://forum.pwnedbox.site/
**Affected Feature:** Edit Profile -> Profile Signature (`Signature` field)  
**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Profile Signature** feature of the Acronis Forum web application (`https://forum.pwnedbox.site`). As indicated by the **Live Preview (rendered as HTML)** box beneath the input field, user-supplied input in the **Signature** field is rendered as raw HTML without proper server-side sanitization or output encoding.

When a user saves a malicious HTML payload in their signature, it is persisted in the backend database and executed automatically in the browser of any user or forum administrator who views that profile or any forum thread/post displaying the user's signature.

## Steps to Reproduce
## Steps to Reproduce
1. Log in to `https://forum.pwnedbox.site` and navigate to **My Profile** (**Edit Profile**).
2. Fill in the required fields (**Display Name** and **About Me**) with normal text.
3. Enter `'">abmi1<img src=O onError=alert(1)>` into the **Signature** field and click **Save Profile**.
4. Observe that `alert(1)` executes automatically upon saving and viewing the profile.

## POC (Proof of Concept)
**Navigate to account settings, enter credentials and payload, save**

<img width="1000" height="492" alt="image" src="https://github.com/user-attachments/assets/52eaebb9-1d23-421c-a0e5-10f516715dac" />

**Observe code execution, confirming stored xss**

<img width="1600" height="811" alt="image" src="https://github.com/user-attachments/assets/03e19f6e-958f-4b45-a53e-a44499ded886" />




## Impact
Because forum signatures are displayed across user profiles and public forum threads, this Stored XSS vulnerability can affect any authenticated user, moderator, or administrator who views the attacker's profile or posts. Successful exploitation allows an attacker to:
* Hijack user or moderator/administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as modifying forum content, posting malicious links, or escalating privileges) on behalf of victims.
* Redirect forum visitors to malicious phishing pages under the trusted `forum.pwnedbox.site` domain.

## Remediation
* **Strict Server-Side HTML Sanitization:** If HTML formatting is intentionally supported in profile signatures, sanitize all input on the server side using a strict allowlist-based sanitizer (such as **DOMPurify**, **Bleach**, or **HTML Purifier**) to strip event handlers (`onerror`, `onload`, etc.) and dangerous tags (`<script>`, `<iframe>`, `<object>`).
* **Context-Aware Output Encoding:** If raw HTML is not required in signatures, encode all user-supplied input using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering it on the page.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
