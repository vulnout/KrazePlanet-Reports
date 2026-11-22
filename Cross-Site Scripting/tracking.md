## Title
Reflected XSS - Equifax
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://tracking.pwnedbox.site/

**Affected Parameter:** `search`  
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `search` GET parameter of the web application (`tracking.pwnedbox.site`). Although the application applies HTML entity encoding to angle brackets (`<` and `>`), user-supplied input is reflected directly inside a double-quoted JavaScript string literal within an inline `<script>` block without proper JavaScript string escaping.

Because double quotes (`"`) are not escaped, an attacker can close the enclosing string literal and use JavaScript string concatenation (`+`) to inject and execute arbitrary JavaScript expressions without needing any HTML tags.
## Steps to Reproduce
2. Visit `https://tracking.pwnedbox.site/?search=test%22%2Balert%28document.domain%29%2B%22`
3. The JavaScript payload executes and an alert reflecting the domain name(`tracking.pwnedbox.site`) can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->

**1. visit https://tracking.pwnedbox.site/ and enter payload**

<img width="1600" height="811" alt="image" src="https://github.com/user-attachments/assets/d81da133-d5b0-4dbf-9cc3-eb278b20d6c8" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="823" alt="image" src="https://github.com/user-attachments/assets/2f541875-3737-4c18-83a4-17ff5d5b597c" />

## Impact
An attacker can craft a malicious URL containing the JavaScript-context XSS payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted `tracking.pwnedbox.site` domain.

## Remediation
* **JavaScript Context Encoding:** Apply proper JavaScript string escaping (such as Unicode escaping `\u0022` for `"`, `\u0027` for `'`, and `\u005c` for `\`) to all user-supplied input before interpolating it inside `<script>` blocks—or pass dynamic data safely via HTML `data-*` attributes and parse it with `JSON.parse()` / `element.dataset`.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (avoiding `'unsafe-inline'`) to prevent unauthorized inline script execution.
