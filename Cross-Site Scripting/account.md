## Title
Reflected XSS - Extended Function Filter
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://account.pwnedbox.site/
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameter:** `lname`
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `lname` parameter of the web application. While the application implements a keyword filter that blocks explicit references to common XSS functions such as `alert()` and filters out `<script>`, it fails to properly sanitize or HTML-encode user-supplied input. 

An attacker can bypass this blacklist using JavaScript string concatenation inside `eval()` (e.g., `eval("a"+"lert(2)")`) and mixed-case tag obfuscation (`<sCript>`) to dynamically construct and execute arbitrary JavaScript in the context of a victim's browser session.Though, it is observed that the parameter `fname` successfully filters such attempts.

## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://account.pwnedbox.site/?fname=%27%22%3C%3E%3CscRIpt%3Eeval%28%22a%22%2B%22lert%281%29%22%29%3C%2FscRIpt%3E&lname=%27%22%3C%3E%3CscRIpt%3Eeval%28%22a%22%2B%22lert%281%29%22%29%3C%2FscRIpt%3E`
2. The JavaScript payload executes and an alert `2` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://account.pwnedbox.site/ and enter payload**

<img width="1584" height="815" alt="image" src="https://github.com/user-attachments/assets/d5e50fae-57a6-4cbd-9b4c-a0b977d4ff4f" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="741" alt="image" src="https://github.com/user-attachments/assets/cf00148f-f51d-4a38-a7a1-e773d4bf32d3" />


## Impact
An attacker can craft a malicious URL containing the hidden parameter and payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive keyword blacklists or regex stripping (such as blocking `<script>`) as a primary defense against XSS, as attackers can easily bypass them using mixed casing or alternative event handlers.
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `lname` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTTP response.
* **Avoid Keyword Blacklists:** Do not rely on blocking specific JavaScript function names (like `alert`) as a defense against XSS, as attackers can easily obfuscate payloads using string concatenation, encoding, or alternative execution sinks (`eval`, `Function`, `setTimeout`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (avoiding `'unsafe-inline'` and `'unsafe-eval'`) to prevent inline script execution.
