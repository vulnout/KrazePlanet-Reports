## Title
Reflected XSS - RefSeek: Academic & Scientific Search Engine
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://refseek.pwnedbox.site/

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Affected parameter:** `q`

There is a presence of a Reflected Cross-Site Scripting (XSS) vulnerability in the `q` parameter of the web application's search functionality.User-supplied input passed through the `q` query parameter is reflected directly into the HTTP response without proper HTML encoding or sanitization, allowing an attacker to execute arbitrary JavaScript in the context of a victim's browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://refseek.pwnedbox.site/index.php?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
2. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Visit `https://refseek.pwnedbox.site/` and enter payload**
   
<img width="1600" height="813" alt="image" src="https://github.com/user-attachments/assets/4d70a23c-2241-4cdd-a0be-f12d8b9a10e7" />


**2. Arbitrary JavaScript execution triggered**
   
<img width="1600" height="814" alt="image" src="https://github.com/user-attachments/assets/54c2bb15-e675-40bc-8dc7-7575ec929ae0" />



## Impact
<!-- Explain what an attacker could achieve, who could be affected, and the realistic security consequences. -->
Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.
## Remediation
<!-- Suggest specific steps to fix the vulnerability. Include secure coding practices or configuration changes where appropriate. -->
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `q` parameter using HTML entity encoding (e.g., converting `<` to `&lt;` and `>` to `&gt;`) before rendering it inside the HTML response.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
