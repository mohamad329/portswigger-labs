# LAB 01 - Retrieving Hidden Data

## Lab Information

- **Category:** SQL Injection
- **Difficulty:** Apprentice
- **Status:** ✅ Solved
- **Date:** 2026-6-29

---

## Objective

The goal of this lab is to understand how the invocation condition can be bypassed by using a condition that is always satisfied.

---

## Vulnerability Overview

An SQL injection vulnerability is a type of web vulnerability that targets databases located on the server.

---

## Methodology

1. Reconnaissance
2. Intercept request using Burp Suite
3. Analyze parameters
4. Test for SQL Injection
5. Exploit

---

## Payload Used

```
'OR 1=1--
```

---

## Why It Worked

This payload (SQL injection) succeeds when the application incorporates user input directly into the backend query without sanitization or validation, tricking the database into executing a condition that is always true.
---

## Impact

Extract all data or bypass the login.

---

## Remediation

- Prepared Statements
- Parameterized Queries
- Input Validation
- Least Privilege

---

## Lessons Learned

- 1. Never trust user input.
- 2. Separating data from SQL commands.
- 3. Principle of Least Privilege.
- 4. Suppressing technical error messages.
- 5. Security-centricity and the use of secure libraries
