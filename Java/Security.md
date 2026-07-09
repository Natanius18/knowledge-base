
# Web Security Vulnerabilities Cheat Sheet for Java Developers

## 1. Injection Attacks Overview

Injection vulnerabilities happen when an application sends untrusted user input to an interpreter or execution engine.

Main rule:

> Never trust user input. Validate it, sanitize it, and use safe APIs.

Common injection types:
- SQL Injection
- Code Injection
- Command Injection
- XSS
- LDAP Injection
- Template Injection

---

# Code Injection

## What is Code Injection?

Code Injection happens when an attacker injects code that is interpreted and executed by the application.

Example:
- PHP `eval()`
- JavaScript `eval()`
- Python `exec()`

The attacker controls part of the code executed by the application.

## Why it happens

Cause:
- Dynamic code execution
- Poor input validation
- Using user input inside interpreters

## Prevention

Avoid dynamic execution:

Bad:
```java
eval(userInput);
````

Better:

* Do not execute user-provided code
* Use predefined logic
* Validate input strictly
* Apply least privilege

---

# Command Injection

## What is Command Injection?

Command Injection allows an attacker to execute operating system commands through a vulnerable application.

Example:

```java
Runtime.getRuntime()
       .exec(userInput);
```

Attacker can execute:

```
whoami
cat /etc/passwd
```

## Difference: Code vs Command Injection

| Type              | Executes                  |
| ----------------- | ------------------------- |
| Code Injection    | Application language code |
| Command Injection | OS commands               |

Example:

Code Injection:

```
execute PHP code
```

Command Injection:

```
execute Linux shell command
```

## Prevention

Avoid executing system commands with user input.

Bad:

```java
Runtime.exec("ping " + userInput);
```

Better:

* Use libraries instead of shell commands
* Validate input using allowlists
* Escape arguments
* Run processes with minimum permissions

---

# Server-Side Request Forgery (SSRF)

## What is SSRF?

SSRF happens when an attacker forces the server to make HTTP requests to internal or external resources.

Example:

Application:

```
GET /preview?url=https://example.com
```

Attacker:

```
url=http://localhost/admin
```

Server accesses internal resources.

## Possible impact

Attackers can access:

* Internal services
* Admin panels
* Cloud metadata
* Credentials

Example cloud metadata:

```
169.254.169.254
```

## Prevention

* Do not allow arbitrary URLs
* Validate allowed domains
* Use URL allowlists
* Disable unnecessary protocols
* Disable redirects
* Isolate external requests in a sandbox

---

# Directory Traversal

## What is Directory Traversal?

Allows attackers to access files outside the intended directory.

Example:

Normal:

```
/images/photo.png
```

Attack:

```
../../../../etc/passwd
```

## Risks

Attackers may read:

* Configuration files
* Source code
* Credentials
* Environment files

Example:

```
application.properties
.env
.git
```

## Prevention

Never directly use user input as file paths.

Bad:

```java
new File(userInput);
```

Better:

```java
Path path = basePath.resolve(userInput)
                    .normalize();

if (!path.startsWith(basePath)) {
    throw new SecurityException();
}
```

Also:

* Store secrets outside web root
* Use allowlists
* Avoid exposing file systems

---

# Broken Access Control

## What is Access Control?

Access control defines who can perform which actions.

Main principle:

> Authentication tells who you are. Authorization tells what you can do.

---

## Types of Access Control

### Vertical Privilege Escalation

User gains higher privileges.

Example:

User → Admin

```
/admin/deleteUser
```

without permission check.

---

### Horizontal Privilege Escalation

User accesses another user's resources.

Example:

```
GET /account?id=123
```

Changing:

```
GET /account?id=456
```

reveals another user data.

---

### IDOR (Insecure Direct Object Reference)

A common access control mistake.

Bad:

```java
@GetMapping("/users/{id}")
User getUser(@PathVariable Long id)
```

without checking ownership.

Good:

```java
checkUserPermission(currentUser, requestedUser);
```

---

## Prevention

* Deny access by default
* Check authorization on every request
* Never rely on frontend restrictions
* Implement role-based access control
* Validate object ownership
* Use centralized security rules

Spring example:

```java
@PreAuthorize("hasRole('ADMIN')")
deleteUser();
```

---

# Cross-Site Scripting (XSS)

## What is XSS?

XSS happens when attacker-controlled JavaScript is executed in another user's browser.

Example:

User comment:

```html
<script>alert('hack')</script>
```

---

## Types of XSS

### Reflected XSS

Payload comes from request.

Example:

```
/search?q=<script>alert(1)</script>
```

---

### Stored XSS

Payload is stored permanently.

Example:

Database:

```
comment = <script>stealCookie()</script>
```

Every visitor executes it.

---

### DOM-based XSS

Client-side JavaScript writes unsafe data into HTML.

Dangerous APIs:

```javascript
innerHTML
document.write()
```

---

## Prevention

Backend:

* Validate input
* Encode output
* Use safe templates

Frontend:

* Avoid `innerHTML`
* Use frameworks escaping by default

Spring:

* Use Thymeleaf escaping:

```html
${userInput}
```

instead of raw HTML rendering.

---

# Cross-Site Request Forgery (CSRF)

## What is CSRF?

CSRF tricks an authenticated user into performing unwanted actions.

Example:

Victim is logged in:

```
bank.com
```

Attacker sends:

```
transfer money request
```

Browser automatically sends cookies.

---

## Requirements

CSRF needs:

1. Victim is authenticated
2. Browser automatically sends credentials
3. No CSRF protection exists

---

## Prevention

Use:

### CSRF Tokens

Example:

```
POST /change-password

csrfToken=abc123
```

Server verifies token.

---

### SameSite Cookies

Example:

```
SameSite=Strict
```

Prevents cross-site cookie sending.

---

# CORS

## What is CORS?

CORS controls which domains can access resources from another domain.

Example:

Allowed:

```
frontend.com
        |
        v
api.com
```

---

## Common mistake

Bad:

```
Access-Control-Allow-Origin: *
```

with credentials.

---

## Prevention

* Allow only trusted origins
* Avoid wildcard with authentication
* Validate CORS configuration

Example:

```
Access-Control-Allow-Origin:
https://myfrontend.com
```

---

# Clickjacking

## What is Clickjacking?

User clicks something different from what they see.

Usually done with invisible iframes.

Example:

Fake button:

```
Click here to win prize
```

Actually clicks:

```
Delete account
```

---

## Prevention

Use headers:

```
X-Frame-Options: DENY
```

or CSP:

```
Content-Security-Policy:
frame-ancestors 'none';
```

---

# Open Redirect

## What is Open Redirect?

Application redirects users to attacker-controlled URLs.

Example:

```
/login?redirect=https://evil.com
```

User trusts the domain but gets redirected.

---

## Risks

* Phishing
* OAuth attacks
* Credential theft

---

## Prevention

Avoid dynamic redirects.

Bad:

```java
redirect(userUrl);
```

Better:

Allow only internal paths:

```
/profile
/settings
/dashboard
```

Validate redirect destinations.

---

# Security Rules Every Java Developer Should Remember

## Input Validation

* Never trust user input
* Prefer allowlists over blocklists
* Validate data type, length, format

---

## Authentication vs Authorization

Authentication:

"Who are you?"

Authorization:

"What can you do?"

Always check authorization.

---

## Use Secure APIs

Avoid:

* Dynamic SQL
* Shell commands
* Reflection with user input
* Dynamic code execution

Prefer:

* Prepared statements
* ORM safely (JPA/Hibernate)
* Framework security features

---

## Secrets Management

Never store:

```
password=123456
apiKey=xxxx
```

inside:

```
application.properties
git repository
docker image
```

Use:

* Environment variables
* Secret managers
* Vault solutions

---

## Logging

Log:

* Authentication failures
* Authorization failures
* Suspicious activity

Do not log:

* Passwords
* Tokens
* Personal secrets




