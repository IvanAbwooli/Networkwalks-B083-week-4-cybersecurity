# Mediroza General Hospital – Penetration Testing Project

## 📌 Project Overview

This project documents an authorized penetration-testing exercise conducted against the **Mediroza General Hospital** web application as part of a cybersecurity/network security practical.

The assessment focuses on identifying common web application security weaknesses, demonstrating their impact in a controlled environment, and documenting appropriate mitigation measures.

> **⚠️ Disclaimer:** This project is intended strictly for an authorized educational/laboratory environment. Testing systems without explicit permission is illegal and unethical.

---

## 🎯 Objectives

The main objectives of the assessment were to:

- Perform reconnaissance against the target web application.
- Identify publicly exposed directories and resources.
- Test the patient login functionality.
- Investigate username enumeration.
- Demonstrate SQL injection against the login mechanism.
- Investigate insecure direct object references (IDOR).
- Identify sensitive information exposed through application functionality.
- Document security findings and recommended mitigations.
- Produce evidence suitable for a penetration-testing report.

---

## 🏥 Target Environment

**Target:** Mediroza General Hospital

**Target URL:**

```text
https://medirozahospital.com/
```

The assessment was performed according to the supplied practical/project scenario and should only be reproduced against the authorized laboratory target.

---

# Milestone 1 – Reconnaissance

## 1.1 Robots.txt Enumeration

The first step was to inspect the site's `robots.txt` file.

### Command

```bash
curl https://medirozahospital.com/robots.txt
```

The assessment identified directories including:

```text
/patient/
/staff/
/old/
```

These entries provided useful information about application areas that may be accessible on the server.

### Security Observation

A `robots.txt` file is **not an access-control mechanism**. Sensitive directories should not rely on `robots.txt` to prevent unauthorized access.

---

# Milestone 2 – Authentication Testing

## 2.1 Patient Login

The patient login application was identified at:

```text
https://medirozahospital.com/patient/login.php
```

The login functionality was tested using the credentials specified in the practical scenario.

Example test account:

```text
Username: bob
Password: test123
```

---

## 2.2 Username Enumeration

Different responses were observed when testing usernames.

For example:

```text
Username: bob
Password: test123
```

returned a response indicating that the username was not found.

A different response was obtained when testing:

```text
Username: admin
Password: test123
```

The application indicated that the password was incorrect.

### Security Impact

Different authentication responses can allow an attacker to determine whether a username exists.

This creates a **username enumeration vulnerability** and can make subsequent password attacks more targeted.

### Recommended Mitigation

The application should return a generic authentication error such as:

```text
Invalid username or password.
```

The response should be identical regardless of whether the username exists.

Additional protections should include:

- Account lockout or throttling.
- Rate limiting.
- Multi-factor authentication.
- Strong password policies.
- Monitoring of repeated authentication failures.

---

# Milestone 3 – SQL Injection Testing

The practical also investigated the login functionality for SQL injection.

The documented test involved entering an SQL injection payload into the username field rather than treating the credentials shown in the report as a terminal command.

### Security Issue

If user input is directly concatenated into an SQL query, specially crafted input may alter the intended SQL statement.

A vulnerable query could conceptually resemble:

```sql
SELECT * FROM users
WHERE username = '<user_input>'
AND password = '<password>';
```

If the application does not properly parameterize the query, the input may change the logic of the SQL statement.

---

## Impact

Successful SQL injection can potentially result in:

- Authentication bypass.
- Unauthorized database access.
- Disclosure of sensitive information.
- Modification or deletion of database records.
- Further compromise of the application.

---

## Recommended Mitigation

The application should use:

### Parameterized Queries

Instead of constructing SQL statements through string concatenation, use prepared/parameterized statements.

For example:

```python
cursor.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

The exact syntax depends on the programming language and database driver.

Additional controls should include:

- Input validation.
- Prepared statements.
- Least-privilege database accounts.
- Secure password hashing.
- Error-message suppression.
- Regular security testing.

---

# Milestone 4 – IDOR Testing

The assessment also investigated the possibility of an **Insecure Direct Object Reference (IDOR)** vulnerability.

IDOR occurs when an application exposes an internal object identifier and fails to verify whether the currently authenticated user is authorized to access that object.

For example, an application might expose a resource through a URL containing an identifier:

```text
/resource.php?id=123
```

If changing the identifier allows access to another user's information without an authorization check, the application may be vulnerable to IDOR.

---

## Security Impact

An IDOR vulnerability can expose:

- Patient records.
- Staff information.
- Medical information.
- Internal application data.
- Other users' private resources.

For a hospital environment, unauthorized access to patient information can have particularly serious privacy implications.

---

## Recommended Mitigation

Applications should perform **server-side authorization checks** for every protected resource.

The server should verify:

```text
Authenticated user
        ↓
Requested resource
        ↓
Authorization check
        ↓
Allow / Deny
```

Changing an object identifier must never be sufficient to gain access to another user's information.

---

# Findings Summary

| Finding                                  | Category               | Potential Impact                            | Recommended Control                           |
| ---------------------------------------- | ---------------------- | ------------------------------------------- | --------------------------------------------- |
| Exposed directories through `robots.txt` | Information Disclosure | Reveals application paths                   | Review exposed paths and server configuration |
| Username enumeration                     | Authentication         | Helps identify valid accounts               | Generic login errors and rate limiting        |
| SQL Injection                            | Injection              | Possible authentication/database compromise | Parameterized queries                         |
| IDOR                                     | Broken Access Control  | Possible unauthorized data access           | Server-side authorization                     |
| Sensitive information exposure           | Information Disclosure | Privacy/data leakage                        | Access control and data minimization          |

---

# Risk Considerations

The identified weaknesses demonstrate several important classes of web application vulnerabilities:

1. **Information Disclosure**
2. **Authentication Weaknesses**
3. **Injection**
4. **Broken Access Control**

These vulnerabilities can become more serious when combined. For example, information obtained during reconnaissance can assist authentication attacks, while an authentication or authorization weakness can expose sensitive application data.

---

# Recommended Security Improvements

## Authentication

- Use generic authentication error messages.
- Implement rate limiting.
- Enable multi-factor authentication.
- Enforce strong passwords.
- Monitor failed login attempts.

## Database Security

- Use prepared statements.
- Never construct SQL queries directly from untrusted input.
- Apply least-privilege database permissions.
- Store passwords using strong password-hashing algorithms.

## Access Control

- Perform authorization checks on every protected resource.
- Do not rely on object IDs alone.
- Validate ownership of requested records.
- Implement role-based access control.

## Information Disclosure

- Avoid exposing unnecessary application paths.
- Review `robots.txt` contents.
- Remove debug information from production systems.
- Configure secure error handling.

---

# Tools Used

The assessment can be performed using standard security-testing tools available in Kali Linux, including:

```text
Kali Linux
curl
Web browser
Burp Suite
Nmap
```

The exact tool should be selected according to the individual test being performed.

---

# Example Reconnaissance Command

```bash
curl https://medirozahospital.com/robots.txt
```

For local lab work, replace the target with the authorized laboratory host.

---

# Evidence and Documentation

Screenshots and command outputs should be added to the project repository to demonstrate the testing p

---

# Ethical and Legal Considerations

All penetration-testing activities must be performed only against systems for which explicit authorization has been obtained.

The techniques documented in this repository are intended for:

- Cybersecurity education.
- Authorized penetration testing.
- Controlled laboratory environments.
- Security research with permission.

Do **not** use these techniques against hospital systems, websites, servers, accounts, or networks without authorization.

---

# Conclusion

The Mediroza General Hospital practical demonstrates how a penetration tester can systematically investigate a web application, identify security weaknesses, assess their potential impact, and recommend appropriate controls.

The exercise covers important web-security concepts including reconnaissance, username enumeration, SQL injection, and insecure direct object references.

The primary lesson is that secure web applications require **defense in depth**. Authentication, authorization, input handling, database security, and information disclosure controls must all be implemented correctly rather than relying on a single security mechanism.

---

## 📚 Project Information

**Project:** Mediroza General Hospital Penetration Testing

**Platform:** Kali Linux

**Assessment Type:** Authorized Web Application Security Assessment

**Purpose:** Cybersecurity Education / Practical Lab

**Documentation:** GitHub README

---

## ⚠️ Responsible Use

This repository is provided for educational and authorized security-testing purposes only. The author does not encourage unauthorized access, data theft, credential attacks, or disruption of systems.

Always obtain written permission before conducting penetration testing on systems you do not own.
