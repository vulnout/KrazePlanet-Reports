## Title
Reflected XSS - PUBG
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://assets.pwnedbox.site/

**Affected Parameter:** `p`
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `p` parameter of the web application's search functionaltiy. While the application implements a keyword filter that blocks explicit references to common XSS functions such as `alert()`, it fails to properly sanitize or HTML-encode user-supplied input. 

An attacker can bypass this blacklist using JavaScript string concatenation inside `eval()` (e.g., `eval("a"+"lert(1)")`) to dynamically construct and execute arbitrary JavaScript in the context of a victim's browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://assets.pwnedbox.site/?p=%27%22%3C%3E%3Cscript%3Eeval%28%22a%22%2B%22lert%281%29%22%29%3C%2Fscript%3E`
2. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://assets.pwnedbox.site/ and enter payload**

<img width="1600" height="813" alt="image" src="https://github.com/user-attachments/assets/ab1cbd38-8885-4be2-8226-085b06a1e166" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="813" alt="image" src="https://github.com/user-attachments/assets/6d0cb788-9604-4d61-afc9-f3087c27ea13" />


## Impact
An attacker can craft a malicious URL containing the hidden parameter and payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `p` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTTP response.
* **Avoid Keyword Blacklists:** Do not rely on blocking specific JavaScript function names (like `alert`) as a defense against XSS, as attackers can easily obfuscate payloads using string concatenation, encoding, or alternative execution sinks (`eval`, `Function`, `setTimeout`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (avoiding `'unsafe-inline'` and `'unsafe-eval'`) to prevent inline script execution.
