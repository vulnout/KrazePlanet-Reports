## Title
Reflected XSS - Search Filter Bypass
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://kb.pwnedbox.site/
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Parameter:** `search`  

A **Reflected Cross-Site Scripting (XSS)** vulnerability exists in the `search` GET parameter of the KrazePlanetLabs Knowledge Base & Search application (`kb.pwnedbox.site`). User-supplied input submitted through the search bar is reflected inside an HTML attribute without proper context-aware output encoding.

While the application implements multiple input filters—blocking standard lowercase `<script>` tags and literal references to the `alert` keyword—an attacker can chain three bypass techniques to achieve arbitrary JavaScript execution:
1. **HTML Attribute Breakout (`'">`):** Escapes the enclosing HTML attribute context.
2. **Mixed-Case Tag Obfuscation (`<ScRiPt>`):** Evades case-sensitive tag blacklists.
3. **JavaScript Unicode Identifier Escapes (`\u0061\u006c\u0065\u0072\u0074`):** Evades static keyword filters blocking `alert` while resolving to `alert(1)` at runtime.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://kb.pwnedbox.site/?search=%27%22%3E%3CScRiPt%3E%5Cu0061%5Cu006c%5Cu0065%5Cu0072%5Cu0074%281%29%3C%2FScRiPt%3E`
2. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://kb.pwnedbox.site/ and enter payload**

<img width="1599" height="809" alt="image" src="https://github.com/user-attachments/assets/746d2492-6a0e-4875-b61e-cc3e6775d93b" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="689" alt="image" src="https://github.com/user-attachments/assets/667496b3-111b-4849-92c4-9079e26aabca" />

## Impact
An attacker can craft a malicious URL containing the obfuscated search query and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted `kb.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `search` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive tag filters or keyword blacklists (such as blocking `<script>` or `alert`) as a defense against XSS, as attackers can easily bypass them using mixed casing and JavaScript Unicode escape sequences (`\uXXXX`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
