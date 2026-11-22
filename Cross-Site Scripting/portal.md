## Title
Reflected XSS - String Concatenation Bypass
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://portal.pwnedbox.site/

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameter:** `cat`

A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `cat` GET parameter of the web application (`portal.pwnedbox.site`). User-supplied input is reflected inside an HTML attribute without proper context-aware output encoding.

While the application implements multiple input filters—blocking standard lowercase `<script>` tags and literal references to the `alert` keyword—an attacker can chain three bypass techniques to achieve arbitrary JavaScript execution:
1. **HTML Attribute Breakout (`'">`):** Escapes the enclosing HTML attribute context.
2. **Mixed-Case Tag Obfuscation (`<ScRiPt>`):** Evades case-sensitive tag blacklists.
3. **JavaScript Unicode Identifier Escapes (`\u0061\u006c\u0065\u0072\u0074`):** Evades static keyword filters blocking `alert` while resolving to `alert(1)` at runtime.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Use x8 tool to find hidden parameter `cat`
2. Visit `https://portal.pwnedbox.site/?cat=%27%22%3E%3CScRiPt%3E\u0061\u006c\u0065\u0072\u0074(1)%3C/ScRiPt%3E`
3. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Discover hidden parameter `cat` via x8**

<img width="1600" height="441" alt="image" src="https://github.com/user-attachments/assets/80f3eb1c-f89f-445c-ac02-31c661b862e4" />


**2. visit https://portal.pwnedbox.site/ and enter payload**

<img width="1600" height="815" alt="image" src="https://github.com/user-attachments/assets/0e1cab19-5d2b-418e-96de-7d5d5a1746c3" />


**3. Arbitrary JavaScript execution triggered**

<img width="1600" height="706" alt="image" src="https://github.com/user-attachments/assets/0dadaf24-601e-46d0-b083-0617f948caf5" />



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
