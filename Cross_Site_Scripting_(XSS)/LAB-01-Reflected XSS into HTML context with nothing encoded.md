# LAB XX - Lab Name

## Lab Information

- **Category:** Cross-Site Scripting (XSS)
- **Type:** Reflected / Stored / DOM
- **Difficulty:** Apprentice
- **Status:** ✅ Solved
- **Date:** YYYY-MM-DD

---

![Cross-Site Scripting](https://img.shields.io/badge/Cross_Site_Scripting-XSS-red)

![Solved](https://img.shields.io/badge/Status-Solved-success)

---

# Objective

Describe the purpose of the lab in one or two sentences.

---

# Vulnerability Overview

Explain:

- What is XSS?
- Why does it happen?
- Why is this lab vulnerable?

---

# Attack Prerequisites

Before exploitation, what conditions must exist?

Example:

- User input is reflected in the HTTP response.
- Input is not HTML encoded.
- JavaScript execution is allowed.

---

# Browser Behavior

Explain what the browser does.

Example:

1. Browser receives HTML.
2. Browser parses HTML.
3. Browser encounters a `<script>` tag.
4. Browser executes the JavaScript.

---

# Methodology

1. Browse to the vulnerable page.
2. Identify the search parameter.
3. Test HTML injection.
4. Confirm that HTML is not encoded.
5. Inject JavaScript.
6. Observe successful execution.

---

# Payload Used

```html
<script>alert(1)</script>
```

Explain why this payload works.

---

# Why It Worked

Explain the root technical reason.

Example:

The application reflected user input directly into the HTML response without encoding special characters. When the browser parsed the HTML document, it interpreted the injected `<script>` element as executable JavaScript and executed it.

---

# Exploitation Flow

```
Attacker
      │
      ▼
Inject Payload
      │
      ▼
Web Server
      │
Returns HTML
      │
      ▼
Victim Browser
      │
Parses HTML
      │
      ▼
Executes JavaScript
```

---

# Impact

Explain realistic impact.

Example:

- Session hijacking
- Cookie theft
- Credential theft
- Phishing
- Defacement
- Performing actions as the victim

---

# Root Cause

Explain exactly why the vulnerability exists.

---

# Remediation

- Output Encoding
- Input Validation
- Content Security Policy (CSP)
- HttpOnly Cookies
- Secure Frameworks

Explain each one briefly.

---

# Lessons Learned

Explain what YOU learned.

For example:

- Browsers execute JavaScript inside `<script>` tags.
- HTML encoding prevents browser interpretation.
- XSS targets the client, not the server.
- Understanding HTML context is essential.

---

# Attack Flow Summary

```
Find Reflection
        │
        ▼
Check Encoding
        │
        ▼
Inject HTML
        │
        ▼
Inject JavaScript
        │
        ▼
Browser Executes Code
```

---

# Technical Analysis

Show the response before injection.

```html
You searched for:

TEST
```

Show the response after injection.

```html
You searched for:

<script>alert(1)</script>
```

Explain exactly what changed.

---

# Security Notes

Explain one or two important security concepts learned in this lab.

Example:

The payload executes because the browser trusts the HTML document returned by the server.

---

# Key Takeaways

- Reflected XSS executes immediately.
- HTML Context determines payload behavior.
- Browsers execute JavaScript—not web servers.
- Output encoding is the primary defense.

---

# Screenshots

## Initial Page

...

## Burp Request

...

## Payload Injection

...

## Successful Alert

...
