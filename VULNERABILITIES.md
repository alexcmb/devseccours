# Vulnerabilities

This document lists the security issues visible in `app.py`.

## High Severity

### 1. Hardcoded secrets
- `DATABASE_PASSWORD = "admin123"`
- `SECRET_KEY = "my-secret-key-12345"`

Why it matters:
- Secrets stored in source code can be leaked through Git history, logs, screenshots, or repo access.
- A weak Flask secret key can weaken session integrity and signing.

### 2. SQL injection in `/search`
- The SQL query is built with an f-string using user input:
  - `sql = f"SELECT * FROM transactions WHERE description LIKE '%{query}%'"`

Why it matters:
- An attacker can modify the query structure and read or alter data.
- This can expose all transactions or lead to database compromise.

### 3. Arbitrary Python code execution in `/calculate`
- The route evaluates user input directly:
  - `result = eval(formula)`

Why it matters:
- `eval()` can execute arbitrary Python code.
- This can lead to full remote code execution, file access, process execution, and server takeover.

### 4. Missing CSRF protection on `/add`
- The form accepts POST requests without any anti-CSRF token.

Why it matters:
- A malicious website can cause a logged-in browser session to submit unwanted transactions.
- Even without authentication, this enables cross-site request forgery against any browser session or future auth layer.

## Medium Severity

### 5. Cross-Site Scripting (XSS)
User-controlled values are inserted into HTML without escaping in multiple routes:
- `/search`
- `/calculate`
- `/add`
- `/transactions`
- error pages

Examples:
- `description`, `category`, `query`, `formula`, and exception messages are interpolated directly into HTML.

Why it matters:
- An attacker can inject script content into rendered pages.
- This can steal browser data, modify the page, or perform actions on behalf of the user.

### 6. Information disclosure through debug mode
- The application runs with:
  - `app.run(debug=True, host='0.0.0.0', port=5000)`

Why it matters:
- Debug mode can expose stack traces and internal details.
- Running on `0.0.0.0` makes the service reachable from the network if the port is exposed.

### 7. Detailed error disclosure
- SQL errors and calculation errors are returned directly to the client.
- Exception text and even the SQL query are displayed in the response.

Why it matters:
- Helps attackers understand schema, code paths, and internal behavior.
- Makes exploitation easier.

## Design / Access Control Weaknesses

### 8. No authentication or authorization
- All routes are public:
  - `/`
  - `/search`
  - `/calculate`
  - `/add`
  - `/transactions`

Why it matters:
- Anyone who can reach the app can read and create financial data.
- If this app were used beyond local testing, it would need access control.

### 9. Unsafe HTML generation with string concatenation
- The application builds HTML directly using string concatenation and f-strings.

Why it matters:
- This pattern is error-prone and makes XSS easier to introduce.
- Template escaping is bypassed entirely.

## Operational Risk

### 10. Database reset on every start
- The database is reinitialized on startup:
  - `init_db()` is called both when the file does not exist and when it does exist.

Why it matters:
- This causes data loss.
- While not a classic security issue by itself, it is dangerous in any environment with real data.

## Summary

The most critical issues are:
1. `eval()` remote code execution
2. SQL injection
3. Hardcoded secrets
4. Missing escaping leading to XSS
5. Debug mode and verbose error disclosure

Recommended next step: fix these issues in `app.py` before deploying or sharing the app.
