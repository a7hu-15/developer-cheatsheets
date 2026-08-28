# 🔒 Web Application Security & OWASP Top 10 Reference Cheatsheet

A concise developer reference for identifying, preventing, and remediating common web security vulnerabilities.

---

## 🎯 OWASP Top 10 (Quick Reference & Mitigations)

| Vulnerability | Key Risk | Remediation / Defense |
| :--- | :--- | :--- |
| **A01: Broken Access Control** | Users acting outside intended permissions | Enforce server-side RBAC/ABAC; disallow implicit access. |
| **A02: Cryptographic Failures** | Exposed sensitive data in transit or at rest | Use TLS 1.3, AES-256-GCM, and Argon2id/bcrypt for passwords. |
| **A03: Injection (SQLi, Command)** | Untrusted data executed as code | Use parameterized queries (prepared statements) & ORMs. |
| **A04: Insecure Design** | Architectural flaws prior to implementation | Threat modeling, secure design patterns, and unit security tests. |
| **A05: Security Misconfiguration** | Unnecessary features enabled or weak defaults | Harden CORS, disable debug mode, remove default admin credentials. |
| **A06: Vulnerable/Outdated Components** | Exploitation of unpatched 3rd-party libs | Automated dependency audits (`npm audit`, `pip-audit`, Dependabot). |
| **A07: Identification & Auth Failures** | Session hijacking, credential stuffing | Multi-Factor Auth (MFA), secure session tokens, rate limiting. |
| **A08: Software & Data Integrity** | Compromised updates or untrusted deserialization | Validate digital signatures; avoid raw `pickle` or `eval()`. |
| **A09: Logging & Monitoring Failures** | Undetected breaches or insufficient audit trail | Centralized logging, real-time alerts, tamper-proof logs. |
| **A10: Server-Side Request Forgery (SSRF)** | Server fetching malicious internal URLs | Whitelist outgoing domains, disable HTTP redirects, isolate network. |

---

## 💉 1. Preventing Injection (SQLi & XSS)

### Parameterized SQL Queries

❌ **Vulnerable (String Interpolation):**
```python
# NEVER DO THIS
cursor.execute(f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'")
```

✅ **Secure (Prepared Statements):**
```python
# Always use parameterized placeholders
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password_hash))
```

---

### Cross-Site Scripting (XSS) Prevention

- **Reflected & Stored XSS**: HTML-encode output before rendering in browser DOM.
- **DOM-based XSS**: Avoid using `innerHTML`, `document.write()`, or `eval()`. Use `textContent` or DOMPurify.

```javascript
// ❌ Dangerous
element.innerHTML = userInput;

// ✅ Safe DOM assignment
element.textContent = userInput;

// ✅ Safe HTML sanitization
element.innerHTML = DOMPurify.sanitize(userInput);
```

---

## 🛡️ 2. Security HTTP Headers

Enforce modern HTTP security headers in API gateways or web frameworks (Express, FastAPI, Nginx):

```http
# Prevents clickjacking by disabling framing
X-Frame-Options: DENY

# Forces HTTPS connections for 1 year including subdomains
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

# Restricts sources of scripts, styles, and images
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com; object-src 'none';

# Prevents MIME-type sniffing
X-Content-Type-Options: nosniff

# Controls referrer header exposure
Referrer-Policy: strict-origin-when-cross-origin
```

---

## 🔑 3. JWT & Session Management Best Practices

1. **Algorithm Validation**: Explicitly enforce strong signing algorithms (`HS256`, `RS256`). Disallow `none`.
2. **Short Expiration**: Set `exp` claims to short lifetimes (15 min) and use refresh tokens.
3. **Cookie Storage**: Store JWTs in `HttpOnly`, `Secure`, `SameSite=Strict` cookies instead of `localStorage` to mitigate XSS exposure.

```python
# Secure JWT Creation Example (Python PyJWT)
import jwt
import datetime

payload = {
    "sub": user_id,
    "iat": datetime.datetime.now(datetime.timezone.utc),
    "exp": datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(minutes=15)
}

token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
```

---

## 🌐 4. CORS (Cross-Origin Resource Sharing) Setup

❌ **Vulnerable (Wildcard with Credentials):**
```python
# Allows ANY origin to read authenticated responses
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

✅ **Secure Explicit Whitelist:**
```python
# Specify trusted origin domain explicitly
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

---

## 🔐 5. Password Hashing Standards

Never use simple hashing (`MD5`, `SHA1`, `SHA256`) for passwords. Always use adaptive key-stretching functions with unique salts:

```python
import bcrypt

# Hash password
salt = bcrypt.gensalt(rounds=12)
hashed_pw = bcrypt.hashpw(password.encode('utf-8'), salt)

# Verify password
if bcrypt.checkpw(user_input.encode('utf-8'), hashed_pw):
    print("Authentication successful")
```
