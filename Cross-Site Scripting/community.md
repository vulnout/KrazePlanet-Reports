## Title
Stored XSS - Forum Discussions (Reddit Clone)
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://community.pwnedbox.site/

**Affected Parameter / Field:** `Organization / Institution` (Registration Form)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Organization / Institution** field of the user registration form on `https://community.pwnedbox.site`. User-supplied input entered during account registration is stored in the backend database and rendered unescaped inside the **User Profile** page  under **Contact & Affiliation**.

Because the application fails to perform proper HTML entity encoding on this profile attribute, any user or administrator who views the affected user profile will have arbitrary JavaScript executed in their browser session.

## Steps to Reproduce
1. Navigate to `https://community.pwnedbox.site` and open the **Register** page.
2. Fill in the required account fields, enter `'">abmi5<img src=O onError=alert(5)>` into the **Organization / Institution** field, and submit the registration.
3. Log in to the newly created account via `https://community.pwnedbox.site/index.php?view=login`.
4. Click **My Profile** and observe that `alert(5)` executes automatically on page load.

## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Register new account and enter payload in organization/institution form**

<img width="1600" height="817" alt="image" src="https://github.com/user-attachments/assets/251b75e5-3237-4660-9fbe-c36e74130393" />

**2. Log in into account**

<img width="1600" height="807" alt="image" src="https://github.com/user-attachments/assets/8f2e9b1f-9f3a-452a-8a85-2dcd68bb60e2" />

**3. Go to Profile**

<img width="1599" height="811" alt="image" src="https://github.com/user-attachments/assets/2f90ad1f-7965-413f-8be0-5bc596fd27b6" />

**4. Observe Javascript execution**

<img width="1600" height="816" alt="image" src="https://github.com/user-attachments/assets/338529ec-bc24-4629-a03a-86893b89cf6c" />

## Impact
Because profile details are stored persistently and rendered without sanitization, any user, moderator, or administrator who views the attacker's profile (including admins reviewing user registration details) will automatically execute the injected JavaScript. Successful exploitation allows an attacker to:
* Hijack user or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as modifying community posts, changing account settings, or escalating privileges) on behalf of victims.
* Redirect visitors to malicious phishing pages under the trusted `https://community.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied registration and profile fields (including `Organization / Institution`) using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them inside HTML responses.
* **Input Validation:** Restrict organization/institution names to expected alphanumeric and punctuation characters on the server side.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline event handlers from executing.
