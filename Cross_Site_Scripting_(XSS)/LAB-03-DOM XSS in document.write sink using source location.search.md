# LAB 03 - DOM XSS in document.write sink using source location.search

## Lab Information

- **Category:** Cross-Site Scripting (XSS)
- **Type:** DOM-based 
- **Difficulty:** Apprentice
- **Status:** ✅ Solved
- **Date:** 2026-8-6

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

In this lab, a DOM based XSS vulnerability was exploited by closing the image's existing attributes and injecting a new attribute containing an `onerror` event handler to trigger the payload a technique known as exploiting an HTML attribute context.
Exploiting DOM based XSS allows for the execution of malicious JavaScript within the victim's browser, granting the attacker full control over what the user sees or does on the page; this can lead to session theft, page content modification, redirection, and the execution of actions on the user's behalf. In this specific lab, I demonstrated this control by displaying a message using the `alert()` function.

---

# Objective

The goal of this lab is to exploit a DOM based XSS vulnerability, where the payload is stored within the JavaScript code executed by the browser.

---

# Vulnerability Overview

A DOM-based XSS (Document Object Model-based Cross-Site Scripting) vulnerability is a type of security flaw that occurs entirely within the user's browser on the client side. It happens when JavaScript code takes data from an attacker-controllable source and passes it—unsanitized—to a dangerous function that executes code (a "sink").
The vulnerability arises when three key elements are present in the page's code: the source, the data flow and processing, and the dangerous destination or sink.

---

# Attack Prerequisites

- JavaScript code that reads data modified by an attacker, such as the current page URL (`window.location`), query parameters (`window.location.search`), or the URL hash (`location.hash`).
- A JavaScript function that executes code or injects it into an HTML page—such as `eval()` or `document.write()`—or the insecure use of the `innerHTML` property.
- Transferring data from the source to the executing function without performing any validation or sanitization.

---

# Environment & Scope

| Item | Value |
|------|-------|
| Target | Web page |
| Endpoint | An alert message appears |
| HTTP Method | GET |
| Parameter | Search field |
| Injection Context | HTML |
| Technologies | Browser |

---

# Methodology

- Conducting the test within the research field.
- Launch the page's "Inspect" tool to review the code.
- Determining the injection site.
- Injecting the payload into HTML code.
- Verifying the success of the operation.

---

# Discovery Process

- mohamad
- mohamad ">
- mohamad "> <img src="x" onerror="alert(mohamad)>

---

# Technical Analysis

Show the response before injection.

```html
search:

<img src="/resources/images/tracker.gif?searchTerms=mohamad">

```

Show the response after injection.

```html
search:

<img src="/resources/images/tracker.gif?searchTerms=">
 <img src="x" onerror="alert(mohamad)">
```

---

# Payload Analysis

## Payload Used

```html

"><img src="x" onerror="alert(mohamad)">

```

## Why This Payload Works

It closed the first attribute, then opened a new attribute containing an event; the code within that event triggers if the image fails to load.

---

# Exploitation Flow

```text
Attacker
      │
      ▼
User Input
      │
      ▼
Application
      │
      ▼
Server Processing
      │
      ▼
HTTP Response
      │
      ▼
Browser Rendering
      │
      ▼
Payload Execution
```

---

# Evidence

## html Before injuction

```html
<img src="/resources/images/tracker.gif?searchTerms=mohamad">
```

---

## html After injuction

```http
<img src="/resources/images/tracker.gif?searchTerms=">
 <img src="x" onerror="alert(mohamad)">
```

---

# Impact

- **Data theft:** Accessing authentication cookies or tokens to take control of the victim's account.
- **Performing actions on behalf of the user:** sending messages, purchasing products, or changing the password without the user's knowledge.
- **Redirection:** Redirecting the user to malicious web pages or phishing sites.
- **Modifying page content:** Altering the appearance or text of a page to deceive the user or collect their sensitive information.

---

# Root Cause

A DOM-based XSS vulnerability occurs when client side JavaScript code takes data from attacker-controlled input (known as the "Source") and passes it unsafely to a function or property that executes code in the browser (known as the "Sink"), resulting in the execution of malicious code.

---

# Risk Assessment

| Item | Value |
|------|-------|
| Severity | Medium |
| CVSS Score | 5.9 |
| CWE | CWE-79 |
| OWASP Category | A03:2021-Injection |
| Exploitability | High / Easy |
| Business Impact | Theft of customer accounts and sensitive data , Financial fraud and theft of funds , Reputational damage and loss of trust , Legal Fines and Penalties , Business disruption and incident response costs|

---

# Remediation

- Output Encoding : is a security technique that converts unsafe data into a safe, plain format before displaying it to the user. Its goal is to prevent the browser from executing malicious code and to protect websites against injection attacks, such as Cross-Site Scripting (XSS).
- Input Validation : The process of verifying and confirming that input data meets the required criteria, is secure, and is in the correct format before being processed by the system.
- Content Security Policy (CSP) : A web security standard that enables website owners to control the resources a browser is permitted to load and execute.
- HttpOnly Cookies : A special type of cookie that cannot be accessed by JavaScript code.
- Use templating engines that automatically perform context-aware output encoding.

---

# Lessons Learned

- Browsers execute JavaScript inside `<script>` tags.
- HTML encoding prevents browser interpretation.
- XSS targets the client, not the server.
- Understanding HTML context is essential.

---

# References

- OWASP
- PortSwigger Web Security Academy
- MITRE CWE
- CVSS Specification

---

# Screenshots

## Initial Page

![Test](Screen-Shots/test-lab3.png)

...

## Payload Injection

![Injuction](Screen-Shots/injuct-lab3.png)

...

## Successful Alert

![Success](Screen-Shots/success-lab3.png)

---

# Conclusion

Ultimately, DOM-based XSS is a browser-targeted vulnerability that executes malicious JavaScript code, potentially leading to session data theft or page defacement. This vulnerability typically arises when client-side JavaScript takes data from attacker-controlled input (known as a "Source") and insecurely passes it to a function or property that executes code within the browser (known as a "Sink"), resulting in the execution of malicious code.










