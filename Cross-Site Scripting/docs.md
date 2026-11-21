## Title
Reflected XSS - Page Heading
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://docs.pwnedbox.site/

**Affected Parameter:** `q`

A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in an unlinked/hidden query parameter (`q`) on the web application. Although this parameter is not exposed directly in the user interface, it is still processed by the backend and reflected inside an HTML attribute in the HTTP response. 

Furthermore, while the application implements a case-sensitive filter that blocks standard lowercase `<script>` tags, an attacker can bypass this restriction by combining context-breaking characters (`'"<>`) with mixed-case tag obfuscation (`<sCRipt>`) to execute arbitrary JavaScript in the context of a victim's browser session.


<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->

## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Use x8 tool to find hidden parameter `q`
2. Visit `https://docs.pwnedbox.site/?q=%27%22%3C%3E%3CsCRipt%3Ealert(1)%3C/sCRipt%3E`
3. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Discover hidden parameter `q` via x8**

<img width="1600" height="521" alt="image" src="https://github.com/user-attachments/assets/e88bb8d3-4fd5-4e01-a614-5688d628e504" />

**2. visit https://docs.pwnedbox.site/ and enter payload**

<img width="1598" height="812" alt="image" src="https://github.com/user-attachments/assets/7cad6a64-63a6-476a-9d13-9a0e98b12044" />

**3. Arbitrary JavaScript execution triggered**

<img width="1595" height="804" alt="image" src="https://github.com/user-attachments/assets/a0e9987c-b6b5-4e60-bf62-d36b2c1a86f7" />


## Impact
An attacker can craft a malicious URL containing the hidden parameter and payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `q` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Remove Unused/Debug Parameters:** Audit backend request handlers and remove legacy, hidden, or unused query parameters like `q` if they are no longer required by the application.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive keyword blacklists (such as blocking `<script>`) as a primary defense against XSS, as attackers can easily bypass them using mixed casing or alternative event handlers.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
