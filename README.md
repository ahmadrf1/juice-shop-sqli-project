# OWASP Juice Shop — SQL Injection Authentication Bypass: Discovery, Exploitation & Remediation

## Overview
This project documents a full offensive security engagement against a deliberately
vulnerable application (OWASP Juice Shop), performed in an isolated local lab
(Kali Linux attacker machine, Docker-based target). The engagement follows a
complete vulnerability lifecycle: discovery, exploitation, root cause analysis,
remediation, and retesting.

**Target:** OWASP Juice Shop (`bkimminich/juice-shop`)
**Environment:** Kali Linux + Docker
**Vulnerability Class:** SQL Injection (CWE-89) → Authentication Bypass

## 1. Lab Setup
- Deployed OWASP Juice Shop via Docker on port 3000.
- Extracted the application's compiled source using `docker cp` for static code review.

## 2. Vulnerability Discovery & Exploitation
The login endpoint (`/rest/user/login`) builds its SQL query using direct
string concatenation of user input (see `vulnerable-code/login.js.original`).

**Payload used (email field):** admin@juice-sh.op’--
This breaks out of the intended query structure and comments out the password
check, resulting in authentication bypass as the admin user.

## 3. Root Cause Analysis
File: `build/routes/login.js`

Unsanitized, directly concatenated user input inside a raw SQL string allowed
arbitrary SQL fragments to be injected into the WHERE clause.

## 4. Remediation
The query was rewritten to use parameterized queries via Sequelize's
`replacements` feature (see `fixed-code/login.js.patched`). A patched Docker
image was built using the included `Dockerfile`.

## 5. Retesting
- Deployed the patched image on port 3001 alongside the original (port 3000).
- Re-submitted the identical payload against the patched instance.
- Result: authentication failed as expected (`Invalid email or password`),
  confirming the fix.

## Evidence
Screenshots are available in `screenshots/`, documenting the successful
exploit, the vulnerable code, the failed exploit after the fix, the patched
code, and both containers running side-by-side.

## Key Takeaways
- Direct string concatenation in SQL queries is critical and easily exploitable.
- Parameterized queries fully neutralize this class of injection.
- Verifying a fix requires retesting with the exact exploit payload.

## Disclaimer
This assessment was performed exclusively against OWASP Juice Shop in a fully
isolated local lab. No production systems were involved.
