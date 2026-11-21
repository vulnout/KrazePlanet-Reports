## Title
Reflected XSS - FreshCart: Online Grocery Store

## Description
**Affected URL:** https://freshcart.pwnedbox.site/

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Affected parameter:** `q`

There is a presence of a Reflected Cross-Site Scripting (XSS) vulnerability in the `q` parameter of the web application's search functionality.User-supplied input passed through the `q` query parameter is reflected directly into the HTTP response without proper HTML encoding or sanitization, allowing an attacker to execute arbitrary JavaScript in the context of a victim's browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://freshcart.pwnedbox.site/index.php?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
2. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Visit `https://freshcart.pwnedbox.site/` and enter payload**
   
<img width="1598" height="811" alt="image" src="https://github.com/user-attachments/assets/095b45ab-4d37-4d4d-9065-3723cd473aad" />

**2. Arbitrary JavaScript execution triggered**
   
<img width="1599" height="810" alt="image" src="https://github.com/user-attachments/assets/11b40b8e-208b-4e6a-9c51-d64a468946d1" />


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
