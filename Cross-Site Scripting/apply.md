## Title
Self XSS - POST-Based Reflected XSS
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://apply.pwnedbox.site/

**Affected Input:** `Name` field  

<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
A **Self Cross-Site Scripting (Self-XSS)** vulnerability exists in the **Name** input field on `https://apply.pwnedbox.site`. User-supplied input entered into the form is rendered into the DOM/HTML context without proper output encoding.

While the application implements basic input filters—blocking standard lowercase `<script>` tags and literal references to the `alert` keyword—an attacker (or tricked user) can bypass these restrictions using attribute breakout characters (`'">`), mixed-case tag obfuscation (`<ScRiPt>`), and JavaScript Unicode escape sequences (`\u0061\u006c\u0065\u0072\u0074`) to execute arbitrary JavaScript within their own browser session.
## Steps to Reproduce
<!-- Provide clear, numbered steps so another person can reproduce the issue. Include required requests, parameters, and actions. -->
1. Open a web browser and navigate to `https://apply.pwnedbox.site`.
2. Locate the **What's your name?** form and enter the following payload into the **Name** field:
   ```html
   '"><ScRiPt>\u0061\u006c\u0065\u0072\u0074(1)</ScRiPt>
   ```
3. Click the **Enter** button.
4. Observe that the payload breaks out of the HTML context, bypasses the tag and keyword blacklists, and executes arbitrary JavaScript—triggering an `alert(1)` pop-up.
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. visit https://checkout.pwnedbox.site/ and enter payload**

<img width="1600" height="642" alt="image" src="https://github.com/user-attachments/assets/e8ad767a-8100-49f0-ad5a-8a628341ccd2" />

**2. Arbitrary JavaScript execution triggered**

<img width="1600" height="628" alt="image" src="https://github.com/user-attachments/assets/e8cdf36a-6eb5-4380-811f-f8eeb615ea7e" />

# Impact
Because this vulnerability requires the user to manually enter or paste the payload into their own session (Self-XSS) rather than triggering automatically via a crafted link, direct remote exploitation is limited. However, an attacker can still leverage **social engineering** to trick a victim into pasting the malicious payload into the `Name` field. Successful exploitation allows the attacker to:
* Execute arbitrary JavaScript within the victim's browser session.
* Access sensitive DOM data, local storage, or non-`HttpOnly` session cookies.
* Perform unauthorized actions on behalf of the victim under the `apply.pwnedbox.site` origin.

## Remediation
* **Context-Aware Output Encoding & Safe DOM APIs:** Encode all user-supplied input from the `Name` field using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering, or insert user input safely using `.textContent` instead of `.innerHTML`.
* **Avoid Blacklist-Based Filtering:** Do not rely on case-sensitive tag filters or keyword blacklists (such as blocking `<script>` or `alert`) as a defense against XSS, as they are easily bypassed using mixed casing and Unicode escape sequences (`\uXXXX`).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header to restrict inline script execution and mitigate the impact of XSS.
