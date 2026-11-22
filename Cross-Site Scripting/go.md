## Title
Reflected XSS - Shopify
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://go.pwnedbox.site/

**Affected Parameter:** `returnTo`
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in an unlinked/hidden query parameter (`returnTo`) on the web application. Although this parameter is not exposed directly in the user interface, it is still processed by the backend and reflected inside an HTML attribute in the HTTP response. 

Furthermore, while the application implements a case-sensitive filter that blocks standard lowercase `<script>` tags, an attacker can bypass this restriction by combining context-breaking characters (`'"<>`) with mixed-case tag obfuscation (`<sCRipt>`) to execute arbitrary JavaScript in the context of a victim's browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Use x8 tool to find hidden parameter `returnTo`
2. Visit `https://go.pwnedbox.site/?returnTo=%27%22%3C%3E%3Cscript%3Ealert(1)%3C/script%3E`
3. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Discover hidden parameter `returnTo` via x8**

<img width="1600" height="346" alt="image" src="https://github.com/user-attachments/assets/8fb87038-47cf-4b36-9315-79efc2fd4ff1" />


**2. visit https://docs.pwnedbox.site/ and enter payload**

<img width="1598" height="813" alt="image" src="https://github.com/user-attachments/assets/548d23e8-ce28-4e5e-83d9-105e15c79e88" />

**3. Arbitrary JavaScript execution triggered**

<img width="1600" height="812" alt="image" src="https://github.com/user-attachments/assets/6bd1d110-28a5-45c7-a774-d356c1508b00" />

## Impact
An attacker can craft a malicious URL containing the hidden parameter and payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `returnTo` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Remove Unused/Debug Parameters:** Audit backend request handlers and remove legacy, hidden, or unused query parameters like `returnTo` if they are no longer required by the application.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive keyword blacklists (such as blocking `<script>`) as a primary defense against XSS, as attackers can easily bypass them using mixed casing or alternative event handlers.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
