## Title
Reflected XSS - Multi-Parameter Filter Evasion
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://checkout.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameter:** `cat`

A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `cat` GET parameter of the web application (`checkout.pwnedbox.site`). User-supplied input is reflected inside an HTML attribute without proper context-aware output encoding.

While the application implements multiple input filters—blocking standard lowercase `<script>` tags and literal references to the `alert` keyword—an attacker can chain three bypass techniques to achieve arbitrary JavaScript execution:
1. **HTML Attribute Breakout (`'">`):** Escapes the enclosing HTML attribute context.
2. **Mixed-Case Tag Obfuscation (`<ScRiPt>`):** Evades case-sensitive tag blacklists.
3. **JavaScript Unicode Identifier Escapes (`\u0061\u006c\u0065\u0072\u0074`):** Evades static keyword filters blocking `alert` while resolving to `alert(1)` at runtime.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Use x8 tool to find hidden parameter `cat`
2. Visit `https://checkout.pwnedbox.site/?cat=abmi2%22%3C/title%3E%3CscRIpt%3Ealert(1)%3C/scRIpt%3E`
3. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Discover hidden parameter `cat` via x8**

<img width="1600" height="503" alt="image" src="https://github.com/user-attachments/assets/4496ce16-8e9f-4741-b0f1-618b405002b2" />

**2. visit https://checkout.pwnedbox.site/ and enter payload**

<img width="1600" height="803" alt="image" src="https://github.com/user-attachments/assets/d4375150-9a9f-4202-b83f-e723171a48ac" />

**3. Arbitrary JavaScript execution triggered**

<img width="1600" height="808" alt="image" src="https://github.com/user-attachments/assets/eb8be5b1-4a59-4141-a9b7-58af13c3d079" />


## Impact
An attacker can craft a malicious URL containing the hidden parameter and payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `cat` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Remove Unused/Debug Parameters:** Audit backend request handlers and remove legacy, hidden, or unused query parameters like `cat` if they are no longer required by the application.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive keyword blacklists (such as blocking `<script>`) as a primary defense against XSS, as attackers can easily bypass them using mixed casing or alternative event handlers.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
