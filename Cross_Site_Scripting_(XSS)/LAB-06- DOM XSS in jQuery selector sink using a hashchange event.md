# LAB 06 -  DOM XSS in jQuery selector sink using a hashchange event

## Lab Information

- **Category:** Cross-Site Scripting (XSS)
- **Type:** DOM-based 
- **Difficulty:** Apprentice
- **Status:** ✅ Solved
- **Date:** 2026-8-16

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

A DOM-based Cross-Site Scripting (XSS) vulnerability was identified in this lab. Attacker-controlled input is obtained from the `hashchange` event via `window.location.hash` and passed to a jQuery selector specifically `section.blog-list h2:contains` which acts as a code execution sink.

Since user-supplied values ​​are injected directly into the selector, an attacker can execute JavaScript code by manipulating the page's hash.

---

# Objective

The objective of this lab is to exploit a DOM-based XSS vulnerability caused by the unsafe flow of attacker-controlled data from `window.location.hash` (source) to `section.blog-list h2 : contains` (sink).

---

# Vulnerability Overview

A DOM-based XSS (Document Object Model-based Cross-Site Scripting) vulnerability is a type of security flaw that occurs entirely within the user's browser on the client side. It happens when JavaScript code takes data from an attacker-controllable source and passes it—unsanitized—to a dangerous function that executes code (a "sink").
The vulnerability arises when three key elements are present in the page's code: the source, the data flow and processing, and the dangerous destination or sink In this lab, attacker-controlled input from `window.location.hash` was passed to the jQuery selector `section.blog-list h2 : contains` sink Without any purification.

This allowed the attacker, after reloading the lab page via an `<iframe>` element, to set an `onerror` event containing a `print()` call.

---

# Attack Requirements

- A client-side JavaScript source controllable by the attacker, such as `widow.location.hash`.
- A jQuery selector `section.blog-list h2 : contains` sink that allows attacker-controlled data to influence the URL of a link.
- The absence of effective output encoding or sanitization between the source and the sink.

---

# Environment & Scope

| Item              | Value |
|-------------------|-------|
| Target            | PortSwigger Web Security Academy lab |
| HTTP Method       | GET |
| Source            | `window.location.hash` |
| Sink              | `section.blog-list h2 : contains` |
| Injection Context | HTML Attribute Context |
| Client            | Web Browser |

---

# Methodology

1. Open the vulnerable lab.
2. Inspect the page source and client-side JavaScript.
3. Identify `window.location.hash` as the attacker-controlled source.
4. Trace the data flow to the jQuery selector `section.blog-list h2 : contains` sink.
5. Open the exploit page.
6. Reloading the page using the `<iframe>` element to satisfy the hash requirement.
7. Add an `<img>` element with an `onerror` event that displays the print page.
8. Verify execution by the appearance of the print page.

---

# Discovery Process

### Step 1 — Normal Input

```text
mohamad
```

The search term was passed to the `.attr('href', ...)` destination and interpreted as a link.

### Step 2 — Identify the parameter

```text
?returnPath=/test
```

The `returnPath` parameter controls the value assigned to the `href` attribute of the Back link.

### Step 3 — Confirm DOM Manipulation

```html
<a id="backLink" href="/test">Back</a>
```


### Step 4 — Test JavaScript Scheme

```JavaScript
javascript:alert(1)
```
The `href` attribute was changed to a `javascript:` URI. When the Back link was activated, the browser executed the JavaScript payload.

---

# Technical Analysis

The application reads the query string through `location.search` and passes the resulting value to `.attr('href', ...)`.

The generated HTML initially contains:

```javascript
<script>
$(function() {
    $('#backLink').attr(
        'href',
        (new URLSearchParams(window.location.search)).get('returnPath')
    );
});
 </script>
```

### Before

```html
search:

<a id="backLink" href="/">Back</a>
```

The client-side JavaScript reads the `returnPath` parameter from the URL and assigns its value to the `href` attribute of the Back link.


### After

```html
search:

<a id="backLink" href="javascript:alert(1)">Back</a>
```

After supplying the malicious value, the DOM contains a `javascript:` URI. JavaScript execution occurs when the user activates the link.

---

# Source and Sink Analysis

## Source

```javascript
window.location.search
```
The source is attacker-controlled because the attacker can modify the query string in the URL.

## Parameter

```javascript
returnPath
```

## Data Flow

```javascript
new URLSearchParams(window.location.search)
.get('returnPath')
```

The application extracts the attacker-controlled `returnPath` parameter from the URL query string.

## Sink

```javascript
$('#backLink').attr('href', value);
```
The value is assigned to the `href` attribute of the Back link through jQuery's `.attr()` method.
Because the application does not validate the URL scheme, an attacker can supply a `javascript:` URI. The JavaScript executes when the manipulated link is activated.

---

# Context Analysis

| Property | Value |
|---|---|
| Source | `window.location.search` |
| Parameter | `returnPath` |
| Sink | `.attr('href', ...)` |
| Injection Context | HTML Attribute Context (`href`) |
| Encoding | None |
| Reflection Type | DOM-based |
| URL Scheme Validation | None |
| Payload Type | `javascript:` URI |
| Execution Trigger | User activates the manipulated link |

---

# Payload Analysis

## Payload Used

```javascript

javascript:alert(1)

```

## Why This Payload Works

The payload uses the `javascript:` URI scheme. The application places the attacker-controlled value directly into the `href` attribute without validating the URL scheme.

When the victim activates the manipulated link, the browser interprets the `javascript:` URI and executes the supplied JavaScript code.

---

# Exploitation Flow

```text
Attacker
   │
   ▼
returnPath parameter
   │
   ▼
location.search
   │
   ▼
URLSearchParams
   │
   ▼
returnPath
   │
   ▼
jQuery .attr('href', ...)
   │
   ▼
<a href="javascript:alert(1)">
   │
   ▼
User clicks "Back"
   │
   ▼
JavaScript Execution
```

---

# Evidence

## HTML Before Injection

```html
<a id="backLink" href="/">Back</a>
```

## URL Parameter

```text
?returnPath=javascript:alert(1)
```

---

## HTML After Injection

 ```html
<a id="backLink" href="javascript:alert(1)">Back</a>
 ```

The payload uses the `javascript:` URI scheme. The application places the attacker-controlled value directly into the `href` attribute without validating the URL scheme.

When the victim activates the manipulated link, the browser interprets the `javascript:` URI and executes the supplied JavaScript code.



## Result

```text
User activates the Back link
        ↓
javascript:alert(1)
        ↓
JavaScript execution
        ↓
Alert displayed
```
---

# Impact

- Execute arbitrary JavaScript in the victim's browser context.
- Modify page content and user-visible functionality.
- Perform actions on behalf of the victim.
- Phishing and UI manipulation.
- Access sensitive data exposed to JavaScript.
- Potential session compromise depending on the application's session management and cookie protections.
  
---

# Root Cause

The root cause is the assignment of attacker-controlled URL data to the `href` attribute without validating or restricting the allowed URL schemes.

The application should not allow dangerous schemes such as `javascript:` to reach the DOM.

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

- Validate the `returnPath` value before assigning it to `href`.
- Allow only expected URL schemes such as `https:` and `http:` where appropriate.
- Reject dangerous schemes such as `javascript:` and `data:`.
- Prefer allowlisting known relative paths when the application only needs internal navigation.
- Use safe URL handling and validate the destination before assigning it to `href`.
- Implement a restrictive Content Security Policy (CSP) as an additional defense layer.

  
---

# Lessons Learned

- DOM XSS can occur entirely on the client side.
- `location.search` can act as an attacker-controlled source.
- The `returnPath` parameter controlled the value assigned to the `href` attribute.
- URL scheme validation is critical when user-controlled data is assigned to `href`.
- The `javascript:` URI scheme can result in JavaScript execution when activated.
- Source → Data Flow → Sink analysis is essential for identifying DOM-based XSS.
  
---

# References

- OWASP
- PortSwigger Web Security Academy
- MITRE CWE
- CVSS Specification

---

# Screenshots

## Initial Page

![Test](Screen-Shots/test-lab5.png)

...

## Payload Injection

![Injuction](Screen-Shots/injuct-lab5.png)

...

## Successful Alert

![Success](Screen-Shots/success-lab5.png)

---

# Execution Condition

Unlike some DOM XSS vulnerabilities that execute automatically when the page is loaded, this vulnerability requires the victim to activate the manipulated link.

The payload is stored in the `href` attribute of the Back link and executes when the browser navigates to the `javascript:` URI.

---

# Conclusion

This practical experiment demonstrated how a DOM-based XSS vulnerability can arise when attacker-controlled data is transferred from `location.search` to an insecure sink such as `.attr('href', ...)`.

The key point here is that the success of exploiting an XSS vulnerability depends on identifying the injection context and tracing the data flow path from the source to the sink. In this case, we performed a direct injection (in the URL, since the href attribute accepts many types of URL schemes).
