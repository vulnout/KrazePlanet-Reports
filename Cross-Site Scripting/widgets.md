## Title
Reflected XSS - Reddit
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://widgets.pwnedbox.site/

**Affected Parameter:** `post_id`  
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `post_id` parameter of the web application. User-supplied input is reflected inside an HTML attribute without proper context-aware output encoding.

While the application implements multiple defensive filters—including a leading-character check that blocks payloads starting with special characters (`'`, `"`, `<`), a case-sensitive `<script>` tag filter, and a keyword filter blocking `alert`—an attacker can chain four bypass techniques to achieve arbitrary JavaScript execution:
1. **Alphanumeric Prefix (`abmi`):** Evades naive regex checks that only inspect the start of the input string for special characters.
2. **HTML Attribute Breakout (`'">`):** Escapes the enclosing HTML attribute context.
3. **Mixed-Case Tag Obfuscation (`<ScRiPt>`):** Evades case-sensitive tag blacklists.
4. **JavaScript Unicode Identifier Escapes (`\u0061\u006c\u0065\u0072\u0074`):** Evades static keyword filters blocking `alert` while resolving to `alert(1)` at runtime.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Use x8 tool to find hidden parameter `post_id`
2. Visit `https://widgets.pwnedbox.site/?post_id=abmi%27%22%3E%3CScRiPt%3E\u0061\u006c\u0065\u0072\u0074(1)%3C/ScRiPt%3E`
3. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Discover hidden parameter `post_id` via x8**
<img width="1600" height="526" alt="image" src="https://github.com/user-attachments/assets/26760ba6-5ca6-41c2-adbd-9e1b3b38a2fd" />
**2. visit https://widgets.pwnedbox.site/ and enter payload**
<img width="1596" height="819" alt="image" src="https://github.com/user-attachments/assets/8188a3b8-e701-46c9-af8a-74920b5b35e3" />
**3. Arbitrary JavaScript execution triggered**
<img width="1594" height="788" alt="image" src="https://github.com/user-attachments/assets/52b84075-9287-4ac1-a95d-90bd162ed068" />

## Impact
An attacker can craft a malicious URL containing the obfuscated `post_id` payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Strict Type Validation:** Because `post_id` expects a numeric identifier, enforce strict server-side type casting or regex validation (`^[0-9]+$`) to reject any non-numeric characters immediately.
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `post_id` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Avoid Anchored Regex & Keyword Blacklists:** Do not rely on leading-character regex checks (like `^['"<]`) or case-sensitive keyword blacklists (blocking `<script>` or `alert`), as attackers can easily bypass them with alphanumeric prefixes, mixed casing, and Unicode escape sequences (`\uXXXX`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
