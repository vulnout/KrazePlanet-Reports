## Title
Stored XSS - Bug Bounty Platform (HackerOne Clone)
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://hackerone.pwnedbox.site/
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->

**Affected Feature:** Submit Vulnerability Report (`Title`, `Description`, and `Impact` fields at `index.php?view=submit`)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists across the **Title**, **Description**, and **Impact** fields of the **Submit Vulnerability Report** feature (`https://hackerone.pwnedbox.site`). User-supplied input submitted in a vulnerability report is stored in the backend database and rendered unescaped on the report dashboard without proper HTML entity encoding or sanitization.

Because submitted vulnerability reports are reviewed by triage team members and program administrators, any triager or administrator who opens the report will automatically execute the attacker's arbitrary JavaScript in their privileged browser session.

## Steps to Reproduce
1. Log in to `https://hackerone.pwnedbox.site` and click **Submit report**.
2. Enter `'">abmi5<img src=O onError=alert(1)>` into **Title**, `'">abmi5<img src=O onError=alert(2)>` into **Description**, and `'">abmi5<img src=O onError=alert(3)>` into **Impact**.
3. Select a **Target Program** and **Severity**, then submit the report.
4. View the submitted report on the dashboard and observe that all three stored payloads execute automatically—triggering `alert(1)`, `alert(2)`, and `alert(3)`.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Register new account**

<img width="1600" height="774" alt="image" src="https://github.com/user-attachments/assets/6487566b-6ba4-4b88-87f3-4e0ec0654d69" />

**2. Log in into account**

<img width="1599" height="804" alt="image" src="https://github.com/user-attachments/assets/ca69f50c-82ab-44c4-963a-74b037ba4c22" />

**3. XSS payloads entered into the `Title`, `Description` and `Impact` fields**

<img width="1600" height="817" alt="image" src="https://github.com/user-attachments/assets/73078058-ff4c-4323-8c9c-830fdb112694" />

**4. Observe Javascript execution**

<img width="1600" height="816" alt="image" src="https://github.com/user-attachments/assets/1fad30d2-6dbd-44b0-9857-1200e63bb497" />

<img width="1600" height="811" alt="image" src="https://github.com/user-attachments/assets/6d671e1d-f9ad-4992-b218-c832de9b6ac5" />

<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/9f6ac74f-4e18-4f90-b58a-2c43aa78b54e" />

## Impact
Because vulnerability reports are directly reviewed by platform triagers and program administrators (such as the triage account context), this Stored XSS vulnerability enables **critical privilege escalation and platform account takeover**. When a triager or administrator opens the malicious report, the injected JavaScript executes automatically in their session, allowing an attacker to:
* Hijack triage team or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized administrative actions (such as approving fraudulent bounty payouts, modifying report statuses, or accessing confidential vulnerability reports from other programs) on behalf of the victim.
* Redirect platform users or staff to malicious phishing pages under the trusted `hackerone.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Title`, `Description`, and `Impact` fields using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them on the report dashboard.
* **Strict Server-Side HTML Sanitization:** If Markdown or rich-text formatting is supported in report descriptions, render it safely and sanitize the resulting HTML on the server side using a strict allowlist-based sanitizer (such as **DOMPurify** or **HTML Purifier**) to strip dangerous tags and event handlers (`onError`, `onLoad`, etc.).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
