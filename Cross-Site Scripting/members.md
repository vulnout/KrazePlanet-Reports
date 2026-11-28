## Title
Stored XSS - Product Reviews (Flipkart Clone)
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://members.pwnedbox.site/

**Affected Feature:** Write a Customer Review (`Detailed Review Comments` field)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Detailed Review Comments** field of the product review feature on `https://members.pwnedbox.site`. User-supplied review comments are stored in the backend database and rendered unescaped on the product page without proper HTML entity encoding or sanitization.

Because arbitrary HTML tags and event handlers (such as `<img onError=...>`) are executed directly in the browser, any customer or administrator who views the affected product page will have malicious JavaScript executed in their session.

## Steps to Reproduce
1. Register a new account at `https://members.pwnedbox.siteindex.php?view=register` and log in 
2. Navigate to a product page (e.g., `https://members.pwnedbox.site`) and scroll to **Write a Customer Review**.
3. Select a star rating, enter normal text in **Headline / Short Title**, and enter `'"<img src=O onError=alert(1)>` into **Detailed Review Comments**.
4. Click **Submit Product Review** and observe that `alert(1)` executes automatically upon saving and viewing the product page.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Register new account**

<img width="1600" height="773" alt="image" src="https://github.com/user-attachments/assets/36c6ba6f-7721-448c-9751-3a5f9956c073" />

**2. Log in into account**

<img width="1600" height="815" alt="image" src="https://github.com/user-attachments/assets/fb4f9fe2-85b7-43d9-99af-11e7bd4ce11f" />

**3. Enter XSS payload into the `Detailed Review Comments` field**

<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/7f1e326d-30f2-4440-ace4-512f448c3c66" />

**4. Observe Javascript execution**

<img width="1600" height="792" alt="image" src="https://github.com/user-attachments/assets/689ae367-1512-45eb-9916-4fa624c5d339" />

# Impact
Because customer reviews are publicly visible on product pages, this Stored XSS vulnerability affects every authenticated user, customer, or administrator who visits the compromised product page. Successful exploitation allows an attacker to:
* Hijack user or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as modifying orders, changing account credentials, or escalating privileges) on behalf of victims.
* Inject fake payment forms or redirect shoppers to malicious phishing pages under the trusted `members.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Detailed Review Comments` and `Headline / Short Title` fields using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them on product pages.
* **Server-Side Sanitization:** If basic formatting is required in reviews, sanitize input on the server side using a strict allowlist-based sanitizer (such as **DOMPurify** or **HTML Purifier**) to strip dangerous tags and event handlers (`onError`, `onLoad`, etc.).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
