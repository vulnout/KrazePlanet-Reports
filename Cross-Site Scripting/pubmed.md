## Title
Reflected XSS - PubMed: National Library of Medicine Search Builder
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://pubmed.pwnedbox.site/

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Affected parameter:** `term`

There is a presence of a Reflected Cross-Site Scripting (XSS) vulnerability in the `term` parameter of the web application's search functionality.User-supplied input passed through the `term` query parameter is reflected directly into the HTTP response without proper HTML encoding or sanitization, allowing an attacker to execute arbitrary JavaScript in the context of a victim's browser session.

## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit `https://pubmed.pwnedbox.site/index.php?term=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
2. The JavaScript payload executes and an alert `1` can be seen, confirming reflected XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Visit `https://pubmed.pwnedbox.site/` and enter payload**
   
<img width="1600" height="817" alt="image" src="https://github.com/user-attachments/assets/e1e77d85-49f6-49d9-b630-7f81a8588d3c" />


**2. Arbitrary JavaScript execution triggered**
   
<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/fbf716d0-f6a7-4969-a7b9-64a6c4aac670" />


## Impact
<!-- Explain what an attacker could achieve, who could be affected, and the realistic security consequences. -->
Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.
## Remediation
<!-- Suggest specific steps to fix the vulnerability. Include secure coding practices or configuration changes where appropriate. -->
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `term` parameter using HTML entity encoding (e.g., converting `<` to `&lt;` and `>` to `&gt;`) before rendering it inside the HTML response.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
