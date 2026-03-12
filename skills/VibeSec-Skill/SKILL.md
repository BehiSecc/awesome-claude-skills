---
name: VibeSec-Skill
description: This skill helps Claude write secure web applications. Use this when working on any web application or when a user requests a scan or audit to ensure security best practices are followed.
---

# Secure Coding Guide for Web Applications

## Overview

This guide provides comprehensive secure coding practices for web applications. As an AI assistant, your role is to approach code from a **bug hunter's perspective** and make applications **as secure as possible** without breaking functionality.

**Key Principles:**
- Defense in depth: Never rely on a single security control
- Fail securely: When something fails, fail closed (deny access)
- Least privilege: Grant minimum permissions necessary
- Input validation: Never trust user input, validate everything server-side
- Output encoding: Encode data appropriately for the context it's rendered in

---

## Access Control Issues

Access control vulnerabilities occur when users can access resources or perform actions beyond their intended permissions.

### Core Requirements

For **every data point and action** that requires authentication:

1. **User-Level Authorization**
   - Each user must only access/modify their own data
   - No user should access data from other users or organizations
   - Always verify ownership at the data layer, not just the route level

2. **Use UUIDs Instead of Sequential IDs**
   - Use UUIDv4 or similar non-guessable identifiers
   - Exception: Only use sequential IDs if explicitly requested by user

3. **Account Lifecycle Handling**
   - When a user is removed from an organization: immediately revoke all access tokens and sessions
   - When an account is deleted/deactivated: invalidate all active sessions and API keys
   - Implement token revocation lists or short-lived tokens with refresh mechanisms

### Authorization Checks Checklist

- [ ] Verify user owns the resource on every request (don't trust client-side data)
- [ ] Check organization membership for multi-tenant apps
- [ ] Validate role permissions for role-based actions
- [ ] Re-validate permissions after any privilege change
- [ ] Check parent resource ownership (e.g., if accessing a comment, verify user owns the parent post)

### Common Pitfalls to Avoid

- **IDOR (Insecure Direct Object Reference)**: Always verify the requesting user has permission to access the requested resource ID
- **Privilege Escalation**: Validate role changes server-side; never trust role info from client
- **Horizontal Access**: User A accessing User B's resources with the same privilege level
- **Vertical Access**: Regular user accessing admin functionality
- **Mass Assignment**: Filter which fields users can update; don't blindly accept all request body fields

### Implementation Pattern

```
# Pseudocode for secure resource access
function getResource(resourceId, currentUser):
    resource = database.find(resourceId)
    
    if resource is null:
        return 404  # Don't reveal if resource exists
    
    if resource.ownerId != currentUser.id:
        if not currentUser.hasOrgAccess(resource.orgId):
            return 404  # Return 404, not 403, to prevent enumeration
    
    return resource
```

---

## Client-Side Bugs

### Cross-Site Scripting (XSS)

Every input controllable by the user—whether directly or indirectly—must be sanitized against XSS.

#### Input Sources to Protect

**Direct Inputs:**
- Form fields (email, name, bio, comments, etc.)
- Search queries
- File names during upload
- Rich text editors / WYSIWYG content

**Indirect Inputs:**
- URL parameters and query strings
- URL fragments (hash values)
- HTTP headers used in the application (Referer, User-Agent if displayed)
- Data from third-party APIs displayed to users
- WebSocket messages
- postMessage data from iframes
- LocalStorage/SessionStorage values if rendered

**Often Overlooked:**
- Error messages that reflect user input
- PDF/document generators that accept HTML
- Email templates with user data
- Log viewers in admin panels
- JSON responses rendered as HTML
- SVG file uploads (can contain JavaScript)
- Markdown rendering (if allowing HTML)

#### Protection Strategies

1. **Output Encoding** (Context-Specific)
   - HTML context: HTML entity encode (`<` → `&lt;`)
   - JavaScript context: JavaScript escape
   - URL context: URL encode
   - CSS context: CSS escape
   - Use framework's built-in escaping (React's JSX, Vue's {{ }}, etc.)

2. **Content Security Policy (CSP)**
   ```
   Content-Security-Policy: 
     default-src 'self';
     script-src 'self';
     style-src 'self' 'unsafe-inline';
     img-src 'self' data: https:;
     font-src 'self';
     connect-src 'self' https://api.yourdomain.com;
     frame-ancestors 'none';
     base-uri 'self';
     form-action 'self';
   ```
   - Avoid `'unsafe-inline'` and `'unsafe-eval'` for scripts
   - Use nonces or hashes for inline scripts when necessary
   - Report violations: `report-uri /csp-report`

3. **Input Sanitization**
   - Use established libraries (DOMPurify for HTML)
   - Whitelist allowed tags/attributes for rich text
   - Strip or encode dangerous patterns

4. **Additional Headers**
   - `X-Content-Type-Options: nosniff`
   - `X-Frame-Options: DENY` (or use CSP frame-ancestors)

---

### Cross-Site Request Forgery (CSRF)

Every state-changing endpoint must be protected against CSRF attacks.

#### Endpoints Requiring CSRF Protection

**Authenticated Actions:**
- All POST, PUT, PATCH, DELETE requests
- Any GET request that changes state (fix these to use proper HTTP methods)
- File uploads
- Settings changes
- Payment/transaction endpoints

**Pre-Authentication Actions:**
- Login endpoints (prevent login CSRF)
- Signup endpoints
- Password reset request endpoints
- Password change endpoints
- Email/phone verification endpoints
- OAuth callback endpoints

#### Protection Mechanisms

1. **CSRF Tokens**
   - Generate cryptographically random tokens
   - Tie token to user session
   - Validate on every state-changing request
   - Regenerate after login (prevent session fixation combo)

2. **SameSite Cookies**
   ```
   Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
   ```
   - `Strict`: Cookie never sent cross-site (best security)
   - `Lax`: Cookie sent on top-level navigations (good balance)
   - Always combine with CSRF tokens for defense in depth

3. **Double Submit Cookie Pattern**
   - Send CSRF token in both cookie and request body/header
   - Server validates they match

#### Edge Cases and Common Mistakes

- **Token presence check**: CSRF validation must NOT depend on whether the token is present, always require it
- **Token per form**: Consider unique tokens per form for sensitive operations
- **JSON APIs**: Don't assume JSON content-type prevents CSRF; validate Origin/Referer headers AND use tokens
- **CORS misconfiguration**: Overly permissive CORS can bypass SameSite cookies
- **Subdomains**: CSRF tokens should be scoped because subdomain takeover can lead to CSRF
- **Flash/PDF uploads**: Legacy browser plugins could bypass SameSite
- **GET requests with side effects**: Never perform state changes on GET
- **Token leakage**: Don't include CSRF tokens in URLs
- **Token in URL vs Header**: Prefer custom headers (X-CSRF-Token) over URL parameters


#### Verification Checklist

- [ ] Token is cryptographically random (use secure random generator)
- [ ] Token is tied to user session
- [ ] Token is validated server-side on all state-changing requests
- [ ] Missing token = rejected request
- [ ] Token regenerated on authentication state change
- [ ] SameSite cookie attribute is set
- [ ] Secure and HttpOnly flags on session cookies

---

### Secret Keys and Sensitive Data Exposure

No secrets or sensitive information should be accessible to client-side code.

#### Never Expose in Client-Side Code

**API Keys and Secrets:**
- Third-party API keys (Stripe, AWS, etc.)
- Database connection strings
- JWT signing secrets
- Encryption keys
- OAuth client secrets
- Internal service URLs/credentials

**Sensitive User Data:**
- Full credit card numbers
- Social Security Numbers
- Passwords (even hashed)
- Security questions/answers
- Full phone numbers (mask them: ***-***-1234)
- Sensitive PII that isn't needed for display

**Infrastructure Details:**
- Internal IP addresses
- Database schemas
- Debug information
- Stack traces in production
- Server software versions

#### Where Secrets Hide (Check These!)

- JavaScript bundles (including source maps)
- HTML comments
- Hidden form fields
- Data attributes
- LocalStorage/SessionStorage
- Initial state/hydration data in SSR apps
- Environment variables exposed via build tools (NEXT_PUBLIC_*, REACT_APP_*)

#### Best Practices

1. **Environment Variables**: Store secrets in `.env` files
2. **Server-Side Only**: Make API calls requiring secrets from backend only

---

## Open Redirect

Any endpoint accepting a URL for redirection must be protected against open redirect attacks.

### Protection Strategies

1. **Allowlist Validation**
   ```
   allowed_domains = ['yourdomain.com', 'app.yourdomain.com']
   
   function isValidRedirect(url):
       parsed = parseUrl(url)
       return parsed.hostname in allowed_domains
   ```

2. **Relative URLs Only**
   - Only accept paths (e.g., `/dashboard`) not full URLs
   - Validate the path starts with `/` and doesn't contain `//`

3. **Indirect References**
   - Use a mapping instead of raw URLs: `?redirect=dashboard` → lookup to `/dashboard`

### Bypass Techniques to Block

| Technique | Example | Why It Works |
|-----------|---------|--------------|
| @ symbol | `https://legit.com@evil.com` | Browser navigates to evil.com with legit.com as username |
| Subdomain abuse | `https://legit.com.evil.com` | evil.com owns the subdomain |
| Protocol tricks | `javascript:alert(1)` | XSS via redirect |
| Double URL encoding | `%252f%252fevil.com` | Decodes to `//evil.com` after double decode |
| Backslash | `https://legit.com\@evil.com` | Some parsers normalize `\` to `/` |
| Null byte | `https://legit.com%00.evil.com` | Some parsers truncate at null |
| Tab/newline | `https://legit.com%09.evil.com` | Whitespace confusion |
| Unicode normalization | `https://legіt.com` (Cyrillic і) | IDN homograph attack |
| Data URLs | `data:text/html,<script>...` | Direct payload execution |
| Protocol-relative | `//evil.com` | Uses current page's protocol |
| Fragment abuse | `https://legit.com#@evil.com` | Parsed differently by different libraries |

### IDN Homograph Attack Protection

- Convert URLs to Punycode before validation
- Consider blocking non-ASCII domains entirely for sensitive redirects


---

### Password Security

#### Password Requirements

- Minimum 8 characters (12+ recommended)
- No maximum length (or very high, e.g., 128 chars)
- Allow all characters including special chars
- Don't require specific character types (let users choose strong passwords)

#### Storage

- Use Argon2id, bcrypt, or scrypt
- Never MD5, SHA1, or plain SHA256

---

## Server-Side Bugs

### Server-Side Request Forgery (SSRF)

Any functionality where the server makes requests to URLs provided or influenced by users must be protected.

#### Potential Vulnerable Features

- Webhooks (user provides callback URL)
- URL previews
- PDF generators from URLs
- Image/file fetching from URLs
- Import from URL features
- RSS/feed readers
- API integrations with user-provided endpoints
- Proxy functionality
- HTML to PDF/image converters

#### Protection Strategies

1. **Allowlist Approach** (Preferred)
   - Only allow requests to pre-approved domains
   - Maintain a strict allowlist for integrations

2. **Network Segmentation**
   - Run URL-fetching services in isolated network
   - Block access to internal network, cloud metadata

#### IP and DNS Bypass Techniques to Block

| Technique | Example | Description |
|-----------|---------|-------------|
| Decimal IP | `http://2130706433` | 127.0.0.1 as decimal |
| Octal IP | `http://0177.0.0.1` | Octal representation |
| Hex IP | `http://0x7f.0x0.0x0.0x1` | Hexadecimal |
| IPv6 localhost | `http://[::1]` | IPv6 loopback |
| IPv6 mapped IPv4 | `http://[::ffff:127.0.0.1]` | IPv4-mapped IPv6 |
| Short IPv6 | `http://[::]` | All zeros |
| DNS rebinding | Attacker's DNS returns internal IP | First request resolves to external IP, second to internal |
| CNAME to internal | Attacker domain CNAMEs to internal | DNS points to internal hostname |
| URL parser confusion | `http://attacker.com#@internal` | Different parsing behaviors |
| Redirect chains | External URL redirects to internal | Follow redirects carefully |
| IPv6 scope ID | `http://[fe80::1%25eth0]` | Interface-scoped IPv6 |
| Rare IP formats | `http://127.1` | Shortened IP notation |

#### DNS Rebinding Prevention

1. Resolve DNS before making request
2. Validate resolved IP is not internal
3. Pin the resolved IP for the request (don't re-resolve)
4. Or: Resolve twice with delay, ensure both resolve to same external IP

#### Cloud Metadata Protection

Block access to cloud metadata endpoints:
- AWS: `169.254.169.254`
- GCP: `metadata.google.internal`, `169.254.169.254`, `http://metadata`
- Azure: `169.254.169.254`
- DigitalOcean: `169.254.169.254`

#### Implementation Checklist

- [ ] Validate URL scheme is HTTP/HTTPS only
- [ ] Resolve DNS and validate IP is not private/internal
- [ ] Block cloud metadata IPs explicitly
- [ ] Limit or disable redirect following
- [ ] If following redirects, validate each hop
- [ ] Set timeout on requests
- [ ] Limit response size
- [ ] Use network isolation where possible

---

### Insecure File Upload

File uploads must validate type, content, and size to prevent various attacks.

#### Validation Requirements

**1. File Type Validation**
- Check file extension against allowlist
- Validate magic bytes/file signature match expected type
- Never rely on just one check

**2. File Content Validation**
- Read and verify magic bytes
- For images: attempt to process with image library (detects malformed files)
- For documents: scan for macros, embedded objects
- Check for polyglot files (files valid as multiple types)

**3. File Size Limits**
- Set maximum file size server-side
- Configure web server/proxy limits as well
- Consider per-file-type limits (images smaller than videos)

#### Common Bypasses and Attacks

| Attack | Description | Prevention |
|--------|-------------|------------|
| Extension bypass | `shell.php.jpg` | Check full extension, use allowlist |
| Null byte | `shell.php%00.jpg` | Sanitize filename, check for null bytes |
| Double extension | `shell.jpg.php` | Only allow single extension |
| MIME type spoofing | Set Content-Type to image/jpeg | Validate magic bytes |
| Magic byte injection | Prepend valid magic bytes to malicious file | Check entire file structure, not just header |
| Polyglot files | File valid as both JPEG and JavaScript | Parse file as expected type, reject if invalid |
| SVG with JavaScript | `<svg onload="alert(1)">` | Sanitize SVG or disallow entirely |
| XXE via file upload | Malicious DOCX, XLSX (which are XML) | Disable external entities in parser |
| ZIP slip | `../../../etc/passwd` in archive | Validate extracted paths |
| ImageMagick exploits | Specially crafted images | Keep ImageMagick updated, use policy.xml |
| Filename injection | `; rm -rf /` in filename | Sanitize filenames, use random names |
| Content-type confusion | Browser MIME sniffing | Set `X-Content-Type-Options: nosniff` |

#### Magic Bytes Reference

| Type | Magic Bytes (hex) |
|------|-------------------|
| JPEG | `FF D8 FF` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| GIF | `47 49 46 38` |
| PDF | `25 50 44 46` |
| ZIP | `50 4B 03 04` |
| DOCX/XLSX | `50 4B 03 04` (ZIP-based) |

#### Secure Upload Handling

1. **Rename files**: Use random UUID names, discard original
2. **Store outside webroot**: Or use separate domain for uploads
3. **Serve with correct headers**:
   - `Content-Disposition: attachment` (forces download)
   - `X-Content-Type-Options: nosniff`
   - `Content-Type` matching actual file type
4. **Use CDN/separate domain**: Isolate uploaded content from main app
5. **Set restrictive permissions**: Uploaded files should not be executable

---

### SQL Injection

SQL injection occurs when user input is incorporated into SQL queries without proper handling.

#### Prevention Methods

**1. Parameterized Queries (Prepared Statements)** — PRIMARY DEFENSE
```sql
-- VULNERABLE
query = "SELECT * FROM users WHERE id = " + userId

-- SECURE
query = "SELECT * FROM users WHERE id = ?"
execute(query, [userId])
```

**2. ORM Usage**
- Use ORM methods that automatically parameterize
- Be cautious with raw query methods in ORMs
- Watch for ORM-specific injection points

**3. Input Validation**
- Validate data types (integer should be integer)
- Whitelist allowed values where applicable
- This is defense-in-depth, not primary defense

#### Injection Points to Watch

- WHERE clauses
- ORDER BY clauses (often overlooked—can't use parameters, must whitelist)
- LIMIT/OFFSET values
- Table and column names (can't parameterize—must whitelist)
- INSERT values
- UPDATE SET values
- IN clauses with dynamic lists
- LIKE patterns (also escape wildcards: %, _)

#### Additional Defenses

- **Least privilege**: Database user should have minimum required permissions
- **Disable dangerous functions**: Like `xp_cmdshell` in SQL Server
- **Error handling**: Never expose SQL errors to users

---

### XML External Entity (XXE)

XXE vulnerabilities occur when XML parsers process external entity references in user-supplied XML.

#### Vulnerable Scenarios

**Direct XML Input:**
- SOAP APIs
- XML-RPC
- XML file uploads
- Configuration file parsing
- RSS/Atom feed processing

**Indirect XML:**
- JSON/other format converted to XML server-side
- Office documents (DOCX, XLSX, PPTX are ZIP with XML)
- SVG files (XML-based)
- SAML assertions
- PDF with XFA forms


#### Prevention by Language/Parser

**Java:**
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setExpandEntityReferences(false);
```

**Python (lxml):**
```python
from lxml import etree
parser = etree.XMLParser(resolve_entities=False, no_network=True)
# Or use defusedxml library
```

**PHP:**
```php
libxml_disable_entity_loader(true);
// Or use XMLReader with proper settings
```

**Node.js:**
```javascript
// Use libraries that disable DTD processing by default
// If using libxmljs, set { noent: false, dtdload: false }
```

**.NET:**
```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
settings.XmlResolver = null;
```

#### XXE Prevention Checklist

- [ ] Disable DTD processing entirely if possible
- [ ] Disable external entity resolution
- [ ] Disable external DTD loading
- [ ] Disable XInclude processing
- [ ] Use latest patched XML parser versions
- [ ] Validate/sanitize XML before parsing if DTD needed
- [ ] Consider using JSON instead of XML where possible

---

### Path Traversal

Path traversal vulnerabilities occur when user input controls file paths, allowing access to files outside intended directories.

#### Vulnerable Patterns

```python
# VULNERABLE
file_path = "/uploads/" + user_input
file_path = base_dir + request.params['file']
template = "templates/" + user_provided_template
```

#### Prevention Strategies

**1. Avoid User Input in Paths**
```python
# Instead of using user input directly
# Use indirect references
files = {'report': '/reports/q1.pdf', 'invoice': '/invoices/2024.pdf'}
file_path = files.get(user_input)  # Returns None if invalid
```

**2. Canonicalization and Validation**

```python
import os

def safe_join(base_directory, user_path):
    # Ensure base is absolute and normalized
    base = os.path.abspath(os.path.realpath(base_directory))
    
    # Join and then resolve the result
    target = os.path.abspath(os.path.realpath(os.path.join(base, user_path)))
    
    # Ensure the commonpath is the base directory
    if os.path.commonpath([base, target]) != base:
        raise ValueError("Error!")
    
    return target
```

**3. Input Sanitization**
- Remove or reject `..` sequences
- Remove or reject absolute path indicators (`/`, `C:`)
- Whitelist allowed characters (alphanumeric, dash, underscore)
- Validate file extension if applicable


#### Path Traversal Checklist

- [ ] Never use user input directly in file paths
- [ ] Canonicalize paths and validate against base directory
- [ ] Restrict file extensions if applicable
- [ ] Test with various encoding and bypass techniques

---

## Security Headers Checklist

Include these headers in all responses:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: [see XSS section]
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Cache-Control: no-store (for sensitive pages)
```

---

## JWT Security

JWT misconfigurations can lead to full authentication bypass and token forgery.

### Vulnerabilities

| Vulnerability | Prevention |
|---------------|------------|
| `alg: none` attack | Always verify algorithm server-side, reject `none` |
| Algorithm confusion | Explicitly specify expected algorithm, never derive from token |
| Weak HMAC secrets | Use 256+ bit cryptographically random secrets |
| Missing expiration | Always set `exp` claim |
| Token in localStorage | Store in httpOnly, Secure, SameSite=Strict cookies, never localStorage |


### Secure Implementation

```javascript
// 1. SIGNING
// Always use environment variables for secrets
const secret = process.env.JWT_SECRET; 

const token = jwt.sign({
  sub: userId,
  iat: Math.floor(Date.now() / 1000),
  exp: Math.floor(Date.now() / 1000) + (15 * 60), // 15 mins (Short-lived)
  jti: crypto.randomUUID() // Unique ID for revocation/blacklisting
}, secret, { 
  algorithm: 'HS256' 
});

// 2. SENDING (Cookie Best Practices)
// Protect against XSS and CSRF
res.cookie('token', token, {
  httpOnly: true, 
  secure: true,    
  sameSite: 'strict'
});

// 3. VERIFYING
// CRITICAL: Whitelist the allowed algorithm
jwt.verify(token, secret, { algorithms: ['HS256'] }, (err, decoded) => {
  if (err) {
    // Handle invalid token
  }
  // Trust the payload
});
```

### JWT Checklist

- [ ] Algorithm explicitly specified on verification (never trust token header)
- [ ] `alg: none` rejected
- [ ] Secret is 256+ bits of random data (not a password or phrase)
- [ ] `exp` claim always set and validated
- [ ] Tokens stored in httpOnly cookies (not localStorage/sessionStorage)
- [ ] Refresh token rotation implemented (old refresh token invalidated on use)

---

## API Security

### Mass Assignment

Accepting unfiltered request bodies can lead to privilege escalation.

```javascript
// VULNERABLE — user can set { role: "admin" } in request body
User.update(req.body)

// SECURE — whitelist allowed fields
const allowed = ['name', 'email', 'avatar']
const updates = pick(req.body, allowed)
User.update(updates)
```

This applies to any ORM/framework — always explicitly define which fields a request can modify.

### GraphQL

| Vulnerability | Prevention |
| :--- | :--- |
| Introspection in production | Disable introspection in production environments. |
| Query depth attack | Implement query depth limiting (e.g., maximum of 10 levels). |
| Query complexity attack | Calculate and enforce strict query cost limits. |
| Batching attack | Limit the number of operations allowed per single request. |


```javascript
const server = new ApolloServer({
  introspection: process.env.NODE_ENV !== 'production',
  validationRules: [
    depthLimit(10),
    costAnalysis({ maximumCost: 1000 })
  ]
})
```

---

## General Security Principles

When generating code, always:

1. **Validate all input server-side** — Never trust client-side validation alone
2. **Use parameterized queries** — Never concatenate user input into queries
3. **Encode output contextually** — HTML, JS, URL, CSS contexts need different encoding
4. **Apply authentication checks** — On every endpoint, not just at routing
5. **Apply authorization checks** — Verify the user can access the specific resource
6. **Use secure defaults**
7. **Handle errors securely** — Don't leak stack traces or internal details to users
8. **Keep dependencies updated** — Use tools to track vulnerable dependencies

When unsure, choose the more restrictive/secure option and document the security consideration in comments.
