## Title
Blind Stored XSS - Informatica
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://partners.pwnedbox.site/
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Feature:** User Registration (`Company Name` field at `index.php?view=register`)  
**Vulnerability Type:** Blind / Stored Cross-Site Scripting (XSS)

A **Blind / Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Company Name** field of the user registration form (`https://partners.pwnedbox.site`) on Informatica Cloud. While the `Full Name` field is properly HTML-encoded when displayed in the **Recent Registrations** table, user-supplied input in the `Company Name` field is stored in the backend database and rendered unescaped on both the user dashboard (`index.php?view=dashboard`) and the administrative dashboard (`index.php?view=admin_dashboard`).

Because arbitrary HTML tags and event handlers (such as `<img onError=...>`) execute automatically without sanitization, any administrator who logs in and views the **Recent Registrations** table will immediately execute the attacker's arbitrary JavaScript in their privileged admin session.

## Steps to Reproduce
1. Navigate to `https://partners.pwnedbox.site`.
2. Fill in **Full Name**, **Email Address**, and **Password** with valid details, enter `'">abmi2<img src=O onError=alert(2)>` into the **Company Name** field, and click **Create Account**.
3. Log in to the administrative portal).
4. Observe that upon loading the **Recent Registrations** table on the Admin Dashboard, the stored `Company Name` payload executes automatically—triggering an `alert(2)` pop-up in the administrator's session.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Create account and enter payload in company name field**

<img width="1600" height="804" alt="image" src="https://github.com/user-attachments/assets/d55f1ad6-3211-49b9-9666-54976b6a0e99" />

**2. Log in**

<img width="1600" height="793" alt="image" src="https://github.com/user-attachments/assets/94964c8e-e208-48d3-a135-9ff39ba47d2f" />

**3. Javascript execution when viewed by normal user**

<img width="1600" height="782" alt="image" src="https://github.com/user-attachments/assets/1427d56a-65b0-42a5-bb16-9721031ae52e" />

**4. Javascript execution when viewed by admin**

<img width="1600" height="795" alt="image" src="https://github.com/user-attachments/assets/990b4c7e-a950-46ca-8a28-01f836e1e8e6" />

## Impact
Because registration data (`Company Name`) is displayed directly on the **Admin Dashboard** (`index.php?view=admin_dashboard`) under **Recent Registrations**, this Blind / Stored XSS vulnerability leads directly to **administrative account takeover and full privilege escalation**. When an administrator views the dashboard, the injected JavaScript executes automatically in their privileged context, allowing an attacker to:
* Hijack the administrator's active session by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized administrative actions (such as modifying user roles, deleting accounts, or accessing confidential partner records) on behalf of the admin.
* Redirect administrators to malicious phishing pages under the trusted `partners.pwnedbox.site` domain.

## Remediation
* **Apply Consistent Output Encoding:** Encode user-supplied input from the `Company Name` field using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`)—matching the encoding already applied to the `Full Name` column—before rendering it on user and admin dashboards.
* **Server-Side Input Validation:** Restrict company names to expected alphanumeric and business punctuation characters on the server side to reject HTML tags and event handlers at registration.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
