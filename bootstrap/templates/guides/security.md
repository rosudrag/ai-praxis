# Security Guardrails

Security is **non-negotiable**. These guardrails protect users, data, and systems from harm.

## 1. Secret Detection (CRITICAL)

### Patterns That Must NEVER Appear in Code

Claude must **NEVER** write these patterns to any file:

| Type | Pattern Examples |
|------|------------------|
| API Keys | `sk-...`, `pk_live_...`, `AKIA...`, `ghp_...`, `xox[baprs]-...` |
| Passwords | `password = "..."`, `pwd: "..."`, `secret: "..."` |
| Connection Strings | `mongodb://user:pass@...`, `postgres://...`, `mysql://...` |
| Private Keys | `-----BEGIN RSA PRIVATE KEY-----`, `-----BEGIN OPENSSH PRIVATE KEY-----` |
| Tokens | `Bearer eyJ...`, `token: "..."`, JWT strings |
| Cloud Credentials | `aws_secret_access_key`, `AZURE_CLIENT_SECRET`, `GOOGLE_APPLICATION_CREDENTIALS` inline |

### When User Provides Secrets in Chat

If a user shares a secret in the conversation:

1. **Do NOT echo it back** in responses
2. **Do NOT include it** in any code output
3. **Acknowledge** that you've seen it but won't use it directly
4. **Suggest** the secure alternative immediately

**Example response:**
> "I notice you've shared an API key. I won't include that in the code. Instead, let me set this up using environment variables so your secret stays secure."

### The Secure Pattern

```
# When Claude detects a hardcoded secret, respond:
"I notice this contains [type of secret]. Let me refactor to use environment variables instead."
```

**Always use environment variables or secret managers:**

```python
# Configuration file
import os

DATABASE_URL = os.environ["DATABASE_URL"]
API_KEY = os.environ["API_KEY"]
```

```javascript
// Configuration file
const config = {
  apiKey: process.env.API_KEY,
  dbConnection: process.env.DATABASE_URL,
};
```

```csharp
// Configuration file - use IConfiguration or Secret Manager
public class Settings
{
    public string ApiKey { get; set; } = null!;
}

// In Program.cs
builder.Configuration.AddEnvironmentVariables();
builder.Configuration.AddUserSecrets<Program>(); // Development only
```

### Code Examples

```python
# Configuration
import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env file (never committed)

DATABASE_URL = os.environ["DATABASE_URL"]
API_KEY = os.environ.get("API_KEY", "")  # Optional with default
```

```javascript
// Configuration (Node.js)
require('dotenv').config(); // Load .env file

const config = {
  apiKey: process.env.API_KEY,
  dbUrl: process.env.DATABASE_URL,
};

if (!config.apiKey) {
  throw new Error('API_KEY environment variable is required');
}
```

### Required: .env.example Template

When creating projects with secrets, always include:

```bash
# .env.example - Copy to .env and fill in values
# NEVER commit .env to version control

DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your-api-key-here
SECRET_KEY=generate-a-random-string
```

And ensure `.gitignore` includes:
```
.env
.env.local
.env.*.local
*.pem
*.key
credentials.json
```

---

## 2. Input Validation Requirements

**ALL external input is untrusted.** Validate before use.

### SQL Injection Prevention

```python
# SQL Injection
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE id = " + user_id)

# Parameterized Queries
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
cursor.execute("SELECT * FROM users WHERE id = %(id)s", {"id": user_id})
```

```javascript
// SQL Injection
db.query(`SELECT * FROM users WHERE id = ${userId}`);

// Parameterized Queries
db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

```csharp
// SQL Injection
var query = $"SELECT * FROM Users WHERE Id = {userId}";

// Parameterized Queries (EF Core handles this automatically)
var user = await context.Users.FindAsync(userId);
// Or with raw SQL:
var users = await context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Id = {userId}")
    .ToListAsync();
```

### XSS Prevention

```javascript
// XSS Vulnerable
element.innerHTML = userInput;
document.write(userInput);

// Safe - Escape or use textContent
element.textContent = userInput;
// Or use a sanitization library
element.innerHTML = DOMPurify.sanitize(userInput);
```

```python
# XSS Vulnerable (Flask)
return f"<div>{user_input}</div>"

# Safe - Use template escaping (automatic in Jinja2)
return render_template("page.html", content=user_input)
# Template: <div>{{ content }}</div>  (auto-escaped)
```

### Path Traversal Prevention

```python
# Path Traversal Vulnerable
file_path = f"/uploads/{user_filename}"
open(file_path)  # User could pass "../../../etc/passwd"

# Safe - Validate and resolve paths
import os

base_dir = os.path.abspath("/uploads")
requested_path = os.path.abspath(os.path.join(base_dir, user_filename))

if not requested_path.startswith(base_dir):
    raise ValueError("Invalid file path")

open(requested_path)
```

```javascript
// Path Traversal Vulnerable
const filePath = `./uploads/${userFilename}`;

// Safe - Use path resolution
const path = require('path');
const baseDir = path.resolve('./uploads');
const requestedPath = path.resolve(baseDir, userFilename);

if (!requestedPath.startsWith(baseDir)) {
  throw new Error('Invalid file path');
}
```

### Command Injection Prevention

```python
# Command Injection Vulnerable
os.system(f"convert {user_filename} output.png")
subprocess.call(f"ping {user_input}", shell=True)

# Safe - Use argument arrays, avoid shell=True
import subprocess
import shlex

subprocess.run(["convert", user_filename, "output.png"])
# If shell features needed, validate strictly:
if not user_input.replace(".", "").replace("-", "").isalnum():
    raise ValueError("Invalid input")
```

### Input Validation Checklist

Before using any external input:

- [ ] Type validation (string, number, boolean)
- [ ] Length/size limits enforced
- [ ] Format validation (regex for emails, URLs, etc.)
- [ ] Range validation for numbers
- [ ] Allowlist validation for enumerated values
- [ ] Encoding handled correctly (UTF-8)
- [ ] Null/undefined handled

---

## 3. Authentication & Authorization

### Never Roll Custom Auth

```
# ALWAYS have this discussion before implementing auth:
"I see this requires authentication. Before implementing, let me ask:
1. Is there an existing auth system we should integrate with?
2. Should we use an established library/service (OAuth, Auth0, Firebase Auth)?
3. What are the specific requirements for user sessions?

Rolling custom authentication is risky - I recommend using established solutions."
```

### Recommended Auth Libraries

| Language | Library/Service |
|----------|-----------------|
| JavaScript/Node | Passport.js, Auth0, NextAuth.js, Clerk |
| Python | Flask-Login, Django Auth, Authlib |
| C#/.NET | ASP.NET Identity, IdentityServer, Azure AD |
| Go | Gorilla Sessions, Authboss |
| Ruby | Devise, OmniAuth |
| Java | Spring Security |

### Authentication vs Authorization

**Authentication**: Who are you? (Login)
**Authorization**: What can you do? (Permissions)

```python
# Authentication only (insufficient)
@app.route("/admin/users")
def admin_users():
    if not current_user.is_authenticated:
        return redirect("/login")
    return get_all_users()  # Any logged-in user can access!

# Authentication + Authorization (correct)
@app.route("/admin/users")
def admin_users():
    if not current_user.is_authenticated:
        return redirect("/login")
    if not current_user.has_role("admin"):
        abort(403)  # Forbidden
    return get_all_users()
```

### Session Management Basics

```python
# Secure Session Configuration
app.config.update(
    SESSION_COOKIE_SECURE=True,      # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,    # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',   # CSRF protection
    PERMANENT_SESSION_LIFETIME=3600,  # 1 hour timeout
)
```

```javascript
// Express session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,      // HTTPS only
    httpOnly: true,    // No JavaScript access
    sameSite: 'lax',   // CSRF protection
    maxAge: 3600000,   // 1 hour
  }
}));
```

### Authorization Patterns

```python
# Role-Based Access Control (RBAC)
def requires_role(role):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.has_role(role):
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route("/admin/settings")
@requires_role("admin")
def admin_settings():
    return render_template("admin/settings.html")
```

---

## 4. Dependency Security

### Before Adding Any Package

1. **Check for known vulnerabilities**
   ```bash
   # npm
   npm audit

   # Python
   pip-audit
   safety check

   # .NET
   dotnet list package --vulnerable
   ```

2. **Evaluate the package**
   - When was it last updated?
   - How many maintainers?
   - How many downloads/stars?
   - Are issues being addressed?

3. **Check the license** - is it compatible?

### Package Selection Criteria

| Prefer | Avoid |
|--------|-------|
| Active maintenance (updated within 6 months) | Abandoned projects (no updates in 2+ years) |
| Multiple maintainers | Single maintainer with no activity |
| High download count | Low adoption |
| Good security track record | History of vulnerabilities |
| Clear documentation | Unclear or missing docs |

### Version Pinning

```json
// package.json - Pin exact versions for production
{
  "dependencies": {
    "express": "4.18.2",
    "lodash": "4.17.21"
  }
}
```

```python
# requirements.txt - Pin versions
Django==4.2.7
requests==2.31.0
```

```csharp
<!-- .csproj - Pin versions -->
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
```

### Automated Security Scanning

Set up automated vulnerability scanning:
- **GitHub**: Dependabot alerts
- **npm**: `npm audit` in CI
- **Python**: `safety` or `pip-audit` in CI
- **.NET**: `dotnet list package --vulnerable` in CI

---

## 5. Enforcement Protocol

### Proactive Security Flagging

Claude must **proactively identify and flag** security issues:

```
When security concern detected, use this format:

"**Security Concern**: I've identified [issue type] in [location].

**Risk**: [Explain the potential impact]

**Recommendation**: [Specific fix]

I'll proceed with the secure implementation unless you have specific requirements that prevent this."
```

### Severity Levels and Responses

| Severity | Examples | Claude Response |
|----------|----------|-----------------|
| CRITICAL | Hardcoded secrets, SQL injection | Stop and fix before proceeding |
| HIGH | Missing auth checks, XSS | Flag and fix immediately |
| MEDIUM | Weak session config, missing rate limiting | Flag and recommend fix |
| LOW | Missing security headers, verbose errors | Note for future improvement |

### When to Refuse

Claude must **refuse to proceed** without a security fix for:

1. **Hardcoded secrets** in any form
2. **Direct SQL string concatenation** with user input
3. **Unvalidated file paths** with user input
4. **Shell commands** with unvalidated user input
5. **Custom cryptography** implementation
6. **Disabled security features** without justification

**Refusal language:**
> "I can't implement this as written because it contains [security issue]. This could lead to [consequence]. Let me show you the secure way to do this instead."

### Security Review Checklist

Before considering any code complete:

- [ ] No hardcoded secrets
- [ ] All user input validated
- [ ] Database queries parameterized
- [ ] Output properly encoded/escaped
- [ ] Authentication checks in place
- [ ] Authorization verified (not just authentication)
- [ ] Dependencies scanned for vulnerabilities
- [ ] Error messages don't leak sensitive info
- [ ] Logging doesn't include secrets
- [ ] HTTPS enforced for sensitive operations

---

## Quick Reference Card

### The Golden Rules

1. **Never trust user input** - validate everything
2. **Never hardcode secrets** - use environment variables
3. **Never roll custom crypto** - use established libraries
4. **Never skip authorization** - verify permissions, not just identity
5. **Always use parameterized queries** - prevent SQL injection
6. **Always escape output** - prevent XSS
7. **Always validate paths** - prevent traversal attacks
8. **Always check dependencies** - prevent supply chain attacks

### Response Templates

**Secret detected:**
> "I notice this contains [type]. Let me refactor to use environment variables instead."

**Injection vulnerability:**
> "This code is vulnerable to [type] injection. I'll use parameterized queries/proper escaping instead."

**Missing authorization:**
> "This endpoint needs authorization checks. Currently any authenticated user could access it. Let me add proper permission verification."

**Risky dependency:**
> "Before adding [package], I should note it has [concern]. Consider [alternative] instead, or we can proceed with awareness of the risk."
