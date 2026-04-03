---
name: security-fundamentals
description: Auto-activating security fundamentals skill — vulnerability detection, API key management, JWT validation, SQL injection, XSS scanning, CORS, CSP, and secure coding patterns
origin: jeremylongshore/claude-code-plugins-plus-skills/03-security-fundamentals
tools: Read, Write, Grep, Bash
---

# Security Fundamentals Skill

Automated security assistance: detect vulnerabilities, validate authentication patterns, enforce secure coding standards, and generate security configurations.

## When to Activate

Activates automatically when you mention: security, vulnerability, JWT, API key, SQL injection, XSS, CORS, CSP, authentication, authorization, password, secret, credential, CSRF, rate limiting, input validation, HTTPS, certificate.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `api-key-manager` | Secure API key generation, storage, rotation patterns |
| `jwt-token-validator` | JWT structure, signature verification, expiry checks |
| `oauth2-flow-helper` | Authorization code, PKCE, client credentials flows |
| `sql-injection-detector` | Scan code for unsafe query construction |
| `xss-vulnerability-scanner` | Detect unsanitized output in HTML/JS |
| `code-injection-detector` | Find eval(), exec(), os.system() patterns |
| `hardcoded-credential-finder` | Detect secrets/passwords in source code |
| `env-secret-detector` | Audit .env files and secret management |
| `content-security-policy-generator` | Build strict CSP headers |
| `cors-policy-validator` | Validate CORS configuration for security |
| `csrf-protection-validator` | Check CSRF token implementation |
| `http-header-security-audit` | Audit security headers (HSTS, X-Frame, etc.) |
| `password-strength-analyzer` | Evaluate password policy against NIST guidelines |
| `password-hash-generator` | bcrypt/argon2 hashing patterns |
| `rate-limiter-config` | Token bucket, sliding window rate limiting |
| `input-validation-checker` | Whitelist validation, sanitization patterns |
| `dependency-vulnerability-checker` | npm audit, pip-audit, OWASP dependency-check |
| `https-certificate-checker` | TLS config, certificate validity, cipher suites |
| `security-headers-generator` | Generate complete security header sets |

## Quick Security Checks

```python
# ❌ SQL Injection — NEVER do this
query = f"SELECT * FROM users WHERE name = '{user_input}'"

# ✅ Parameterized query
cursor.execute("SELECT * FROM users WHERE name = %s", (user_input,))

# ❌ Hardcoded credential
API_KEY = "sk-abc123..."

# ✅ Environment variable
import os
API_KEY = os.environ["API_KEY"]  # raises if missing — intentional

# ❌ MD5 for passwords
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()

# ✅ bcrypt
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
```

## JWT Validation Pattern

```python
import jwt

def validate_jwt(token: str, secret: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            secret,
            algorithms=["HS256"],   # explicitly whitelist algorithms
            options={"require": ["exp", "iat", "sub"]}
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise ValueError("Token expired")
    except jwt.InvalidTokenError as e:
        raise ValueError(f"Invalid token: {e}")
```

## Security Headers (Express.js)

```javascript
const helmet = require('helmet');
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true }
}));
```

## Secret Scanning (pre-commit)

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `03-security-fundamentals` (25 skills)
