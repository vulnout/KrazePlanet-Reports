## Title
Reflected XSS - Case-Insensitive Filter Bypass
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://directory.pwnedbox.site/

**Affected Parameter(s):**  `fname` and `lname`
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A Reflected Cross-Site Scripting (XSS) vulnerability exists in the `fname` and `lname` parameter of the web application. While the application implements a case-sensitive filter that blocks standard lowercase `<script>` tags, it fails to properly encode user-supplied input. An attacker can bypass this restriction using mixed-case tag obfuscation (`<sCript>`) to execute arbitrary JavaScript in the context of a victim's browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://directory.pwnedbox.site/?fname=%27%22%3C%3E%3CsCRIpt%3Ealert%281%29%3C%2FsCRIpt%3E&lname=%27%22%3C%3E%3CsCRIpt%3Ealert%282%29%3C%2FsCRIpt%3E`
2. The JavaScript payload executes and alerts `1` and `2` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://directory.pwnedbox.site/ and enter payloads**

<img width="1592" height="811" alt="image" src="https://github.com/user-attachments/assets/50304331-36eb-44c1-ab4b-76b2212db243" />

**2. Arbitrary JavaScript execution triggered**

<img width="1596" height="731" alt="image" src="https://github.com/user-attachments/assets/10a93a39-d0cc-48dc-a5f4-ac8811698bc5" />
<img width="1600" height="768" alt="image" src="https://github.com/user-attachments/assets/1cf8ffd1-d893-41ca-976d-577c3ca6260e" />


## Impact
<!-- Explain what an attacker could achieve, who could be affected, and the realistic security consequences. -->
An attacker can craft a malicious URL containing the payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.


## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `fname` and `lname` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it in the response.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive keyword blacklists or regex stripping (such as blocking `<script>`) as a primary defense against XSS, as attackers can easily bypass them using mixed casing or alternative event handlers.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
