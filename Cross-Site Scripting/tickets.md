## Title
Reflected XSS - Event Handler Filter
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://tickets.pwnedbox.site/

**Affected Parameter:** `lname`
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->

A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `lname` parameter of the web application. While the sibling `fname` parameter properly filters or encodes user input, the `lname` parameter lacks proper context-aware output encoding and relies on a weak keyword/tag blacklist.

An attacker can achieve arbitrary JavaScript execution in the `lname` parameter by combining:
1. **HTML Attribute Breakout (`'">`):** To escape the enclosing HTML attribute context.
2. **Mixed-Case Tag Obfuscation (`<scRIpt>`):** To bypass case-sensitive filters blocking lowercase `<script>` tags.
3. **Bracket Notation & String Concatenation (`window["a"+"lert"](1)`):** To bypass keyword blacklists blocking direct references to `alert`.

## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://tickets.pwnedbox.site/?fname=%27%22%3E%3CscRIpt%3Ewindow%5B%22a%22%2B%22lert%22%5D%281%29%3C%2FscRIpt%3E%22%3E&lname=%27%22%3E%3CscRIpt%3Ewindow%5B%22a%22%2B%22lert%22%5D%282%29%3C%2FscRIpt%3E%22%3E`
2. The JavaScript payload executes and an alert `2` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://tickets.pwnedbox.site/ and enter payload**

<img width="1598" height="812" alt="image" src="https://github.com/user-attachments/assets/5db25bd7-0750-4186-9dbe-b462ccc282ac" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="585" alt="image" src="https://github.com/user-attachments/assets/20088437-6412-42d2-89f2-0ab8b66570c0" />


## Impact
An attacker can craft a malicious link (or auto-submitting CSRF form if sent via POST) targeting the `lname` parameter and trick a victim into interacting with it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Apply Consistent Output Encoding:** Enforce the same strict HTML and attribute entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) on the `lname` parameter as currently implemented on the `fname` parameter.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive tag filters or function-name blacklists (such as blocking `<script>` or `alert`), as attackers can bypass them using mixed casing, string concatenation, and JavaScript bracket notation (`window[...]`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
