## Title
Reflected XSS - Global Site Search Portal
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://search.pwnedbox.site/

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Affected parameter:** `fname` and `lname`

There is a presence of a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameter of the web application. User-supplied input is reflected directly inside an HTML attribute without proper quote or bracket escaping. By prepending context-breaking characters (`'"<>`), an attacker can prematurely close the enclosing attribute and tag to inject and execute arbitrary JavaScript.
## Steps to Reproduce
1. Visit `https://search.pwnedbox.site/?fname=%27%22%3C%3E%3Cscript%3Ealert%281%29%3C%2Fscript%3E&lname=%27%22%3C%3E%3Cscript%3Ealert%282%29%3C%2Fscript%3E`
2. The JavaScript payload executes and alerts `1` and `2` can be seen, confirming reflected XSS execution.

## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Visit `https://search.pwnedbox.site/` and enter payload**

<img width="1577" height="815" alt="image" src="https://github.com/user-attachments/assets/b9b0f25d-7294-45a0-b16c-16365f6d76d6" />

**2. Arbitrary JavaScript execution triggered**

<img width="1592" height="808" alt="image" src="https://github.com/user-attachments/assets/7ed06c3e-4257-4180-a0f6-21d9c45ebbf6" />

<img width="1595" height="811" alt="image" src="https://github.com/user-attachments/assets/31160462-8f44-41ec-bf1e-fd137d47827d" />

## Impact
<!-- Explain what an attacker could achieve, who could be affected, and the realistic security consequences. -->
An attacker can craft a malicious URL containing the payload and trick a victim into clicking it. Successful exploitation allows an attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, tokens, or non-`HttpOnly` session cookies.
* Perform unauthorized actions or redirect the user to phishing pages under the trusted domain.

## Remediation
<!-- Suggest specific steps to fix the vulnerability. Include secure coding practices or configuration changes where appropriate. -->
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `q` parameter using HTML and attribute entity encoding (e.g., converting `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `'` to `&#x27;`) before rendering it inside the HTML response.
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
