# LAB 03 - Querying Database Type and Version (Oracle)

## Lab Information

- **Category:** SQL Injection
- **Difficulty:** Practitioner
- **Status:** ✅ Solved
- **Date:** 2026-7-7

---

![SQL Injection](https://img.shields.io/badge/SQL-Injection-red)

![Solved](https://img.shields.io/badge/Status-Solved-success)

---

## Objective

The objective of this lab is to exploit a UNION-based SQL Injection vulnerability to identify the database type and retrieve the Oracle database version.

---

## Vulnerability Overview

In this lab, a UNION-based SQL Injection was used to combine the original query with a malicious query in order to retrieve information from the Oracle database. Both SELECT statements must return the same number of columns and compatible data types.

---

## Methodology

1. Browse to the vulnerable category page.
2. Intercept the HTTP request using Burp Suite.
3. Identify the vulnerable category parameter.
4. Inject a UNION SELECT payload.
5. Retrieve the Oracle database version from the v$version table.
---

## Payload Used

```sql
' UNION SELECT BANNER,NULL FROM v$version--
```

---

## Why It Worked

The payload terminated the original SQL query and appended a UNION SELECT statement. Because both queries returned the same number of columns with compatible data types, Oracle combined the results and returned the database version from the v$version table.

---

## Impact

An attacker can access information about the database version, enabling them to identify vulnerabilities associated with the specific database type and version.

---

## Root Cause

The application directly concatenated user-controlled input into the SQL query without using parameterized statements, allowing attackers to inject arbitrary SQL commands.

---

## Remediation

- **Prepared Statements**
  - Prevent SQL queries from being modified by user input.

- **Input Validation**
  - Reject unexpected input.

- **Least Privilege**
  - Reduce the impact if SQL Injection occurs.

---

## Lessons Learned

- UNION-based SQL Injection requires the same number of columns.
- Data types of the selected columns must be compatible.
- Oracle stores version information in the v$version table.
- Database fingerprinting is often the first step before further exploitation.

---

## SQL Query Analysis

The application likely executed the following SQL query:

```sql
SELECT name,description
FROM products
WHERE category='Gifts';

```

After SQL injection:

```sql
SELECT name,description
FROM products
WHERE category=''
UNION
SELECT BANNER,NULL
FROM v$version--;

```

---

## Database Specific Notes

- Database: Oracle
- Version Table: v$version
- Version Column: BANNER
  
---

## Screenshots

### Before Exploitation

![Before](screenshots/before-lab3.png)

### Burp Request

![Burp Request](screenshots/burp-request-lab3.png)

### Result

![Result](screenshots/success-lab3.png)
