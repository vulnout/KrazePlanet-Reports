## Title
Stored XSS - Events & Polls Activity Center

## Description
**Affected URL:** https://articles.pwnedbox.site/
<!-- Explain what the vulnerability is, where it exists, and why it is a security issue. Include the affected URL or endpoint. -->
**Affected Feature:** Create Community Event (`Event Title` and `Event Description` fields)  

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in both the **Event Title** and **Event Description** fields of the **Create Community Event** feature on `https://articles.pwnedbox.site`. User-supplied input submitted when publishing an event is stored in the backend database and rendered unescaped under the **Scheduled Events** feed without proper HTML entity encoding or sanitization.

Because arbitrary HTML tags and event handlers (such as `<img onError=...>`) execute automatically whenever the events list loads, any user or administrator who views `https://articles.pwnedbox.site` will have malicious JavaScript executed in their browser session.

## Steps to Reproduce
1. Log in to `https://articles.pwnedbox.site` and navigate to **Events & Polls**.
2. Under **Create Community Event**, enter `'"<img src=O onError=alert(1)>` into **Event Title**, valid text into **Event Date & Time**, and `'"<img src=O onError=alert(2)>` into **Event Description**.
3. Click **Publish Event**.
4. Observe that both stored payloads render in **Scheduled Events** and execute automatically—triggering `alert(1)` followed by `alert(2)`.


## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->
**1. Register new account**

<img width="1600" height="782" alt="image" src="https://github.com/user-attachments/assets/8882f840-cbcc-42fb-91f9-b1f24546350e" />

**2. Log in into account**

<img width="1600" height="772" alt="image" src="https://github.com/user-attachments/assets/19214556-3ffd-4ade-b4c4-2f5571c1437a" />

**3. XSS payloads entered into the `Event Title` and `Event Description` fields**

<img width="1600" height="812" alt="image" src="https://github.com/user-attachments/assets/eeae9ae8-02f8-404e-a5d6-50614485bf7b" />

**4. Observe Javascript execution**

<img width="1600" height="812" alt="image" src="https://github.com/user-attachments/assets/8ab0a90c-76da-491a-abc1-48b2fc3847c4" />

<img width="1600" height="789" alt="image" src="https://github.com/user-attachments/assets/fe9df1a6-15da-449b-b7a0-e29ff4afcba8" />

## Impact
Because published community events are displayed publicly in the **Scheduled Events** feed on the main portal page, this Stored XSS vulnerability affects every authenticated user, member, or administrator who visits `https://pwnedbox.site`. Successful exploitation allows an attacker to:
* Hijack user or administrator sessions by accessing non-`HttpOnly` session cookies or authentication tokens.
* Perform unauthorized actions (such as creating or deleting events/polls, modifying user accounts, or escalating privileges) on behalf of victims.
* Redirect visitors to malicious phishing pages under the trusted `articles.pwnedbox.site` domain.

## Remediation
* **Context-Aware Output Encoding:** Encode all user-supplied input from the `Event Title`, `Event Date & Time`, and `Event Description` fields using HTML entity encoding (`<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, `'` to `&#x27;`) before rendering them in the Scheduled Events feed.
* **Server-Side Sanitization:** If basic HTML formatting is required in event descriptions, sanitize input on the server side using a strict allowlist-based sanitizer (such as **DOMPurify** or **HTML Purifier**) to strip dangerous tags and event handlers (`onError`, `onLoad`, etc.).
* **Implement Content Security Policy (CSP):** Configure a strict `Content-Security-Policy` HTTP response header (disallowing `'unsafe-inline'`) to prevent inline JavaScript execution.
