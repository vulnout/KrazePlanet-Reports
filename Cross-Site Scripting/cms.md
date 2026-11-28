## Title
Stored XSS - Shopify
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://cms.pwnedbox.site/

A Stored Cross-Site Scripting (XSS) vulnerability was identified in the article creation functionality of the website.

The application allows authenticated users to create articles and edit their HTML source through a built-in HTML editor. However, user-supplied HTML content is not properly sanitized or validated before being stored and subsequently rendered.

As a result, an attacker can inject JavaScript-capable HTML into an article. The injected content is persisted by the application and executes when the affected article is rendered in a vulnerable context.

## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Visit https://cms.pwnedbox.site/, create an account, and navigate to the **Write** option.
2. Enter a normal article title, open the **Edit HTML Source** option, and insert the following payload:

   `">abmi1<img src=O onError=alert(1)>`

3. Click **Save** to store the article.
4. Open the saved article and observe the JavaScript alert box, confirming successful Stored XSS execution.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Create a new account**

<img width="1600" height="811" alt="image" src="https://github.com/user-attachments/assets/c567d9b7-4c86-49d3-ae71-fa2ed530ce09" />

**2.Go to write tab**

<img width="1594" height="809" alt="image" src="https://github.com/user-attachments/assets/7bbcd0b4-2503-4f60-a040-3ccb8efc2895" />

**3. Enter normal text as the article title and click edit html source and enter payload**

<img width="1598" height="815" alt="image" src="https://github.com/user-attachments/assets/d5d0b678-110a-4dbe-a396-817e205fc1d1" />

**4.Observe JavaScript execution**

<img width="1600" height="744" alt="image" src="https://github.com/user-attachments/assets/217d063e-7d78-4027-8d51-34be94da10fb" />


## Impact
<!-- Explain what an attacker could achieve, who could be affected, and the realistic security consequences. -->
An attacker could potentially inject client-side scripts into stored article content.

If other users can access the affected article, the vulnerability could allow JavaScript execution in their browsers within the application's origin.

Potential consequences include:
* Manipulation of page content and application UI.
* Phishing through injected content.
* Unauthorized actions performed in a victim's authenticated session, depending on application security controls.

## Remediation
<!-- Suggest specific steps to fix the vulnerability. Include secure coding practices or configuration changes where appropriate. -->
* **HTML Sanitization:** Sanitize user-supplied HTML on the server side before storing or rendering article content. Use a reputable HTML sanitization library with a strict allowlist.
* **Event Handler Filtering:** Remove dangerous event-handler attributes such as `onerror`, `onload`, and other JavaScript-executable attributes from user-controlled HTML.
* **Context-Aware Output Encoding:** Properly encode untrusted input before inserting it into HTML contexts. Avoid interpreting untrusted content as executable markup.
* **Secure HTML Rendering:** Avoid unsafe DOM manipulation methods when rendering user-controlled content.
* **Implement Content Security Policy (CSP):** Configure a restrictive CSP to provide an additional layer of protection against script execution.
