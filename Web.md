# Web Application Penetration Testing Methodology

A comprehensive checklist for conducting web application security assessments. Work through each section systematically — do not skip recon phases to jump to exploitation.

---

## Reconnaissance & Scope Mapping

* [ ] Confirm scope — domains, subdomains, IPs, and any out-of-scope assets
* [ ] Identify all subdomains via passive enumeration (subfinder, amass, dnsx)
* [ ] Perform active subdomain brute-forcing
* [ ] Check DNS records (A, MX, TXT, CNAME, NS) for misconfigurations
* [ ] Identify IP ranges and ASN ownership
* [ ] Check for cloud asset exposure (S3 buckets, Azure Blobs, GCP Storage)
* [ ] Enumerate live hosts and open ports (nmap, httpx)
* [ ] Identify web server software and version via headers and fingerprinting
* [ ] Review robots.txt, sitemap.xml, and .well-known/ endpoints
* [ ] Check for exposed Git repos (/.git/, GitTools, git-dumper)
* [ ] Search for exposed environment files (/.env, /config.yml, /application.properties)
* [ ] Check Wayback Machine / CommonCrawl for historical endpoints
* [ ] Use Google dorks for sensitive file exposure
* [ ] Review job postings and GitHub for technology stack leakage
* [ ] Identify third-party integrations and external JS includes

---

## Technology Fingerprinting

* [ ] Identify frontend framework (React, Angular, Vue, jQuery)
* [ ] Identify backend language and framework (PHP, Python/Django/Flask, Ruby on Rails, Node/Express, Java/Spring)
* [ ] Identify CMS (WordPress, Drupal, Joomla) and check version
* [ ] Identify WAF presence and vendor (wafw00f)
* [ ] Check response headers for technology leakage (X-Powered-By, Server, X-Generator)
* [ ] Review JavaScript files for API endpoints, secrets, and internal tooling references
* [ ] Identify CDN provider and origin IP disclosure
* [ ] Check for GraphQL endpoint (/graphql, /api/graphql) and enable introspection

---

## Authentication

* [ ] Username enumeration via difference in error messages
* [ ] Username enumeration via response timing differences
* [ ] Username enumeration via account lockout behaviour differences
* [ ] Test for default credentials on admin panels and third-party components
* [ ] Test for weak password policy enforcement (minimum length, complexity)
* [ ] Test login form for SQL injection (manual and SQLMap)
* [ ] Test for authentication bypass via parameter manipulation
* [ ] Check if account lockout is enforced after repeated failed attempts
* [ ] Test password reset flow for token predictability or reuse
* [ ] Test password reset links for expiry enforcement
* [ ] Check if password reset tokens are exposed in referrer headers or logs
* [ ] Test for host header injection on password reset emails
* [ ] Check for concurrent session handling issues (multiple simultaneous logins)
* [ ] Test OAuth2 flow for state parameter absence (CSRF on authorisation)
* [ ] Test OAuth2 for redirect_uri bypass and open redirect chaining
* [ ] Check for OAuth2 token leakage via referrer header
* [ ] Test SAML assertions for signature wrapping attacks
* [ ] Test MFA for bypass via response manipulation
* [ ] Test MFA for brute-force protection on OTP codes
* [ ] Check if MFA can be skipped by directly navigating to post-login URLs

---

## Session Management

* [ ] Analyse session token entropy and predictability
* [ ] Check if session token changes on login (session fixation)
* [ ] Check if session token changes on logout
* [ ] Verify session is invalidated server-side on logout
* [ ] Check cookie attributes: Secure, HttpOnly, SameSite
* [ ] Test for session token exposure in URL parameters
* [ ] Check session timeout enforcement (idle and absolute)
* [ ] Test for JWT algorithm confusion (RS256 to HS256)
* [ ] Test for JWT none algorithm acceptance
* [ ] Check JWT claims for privilege escalation opportunities (role, admin, user_id)
* [ ] Test JWT for weak signing secret (brute force with hashcat/jwt_tool)
* [ ] Check if JWT expiry (exp) is enforced server-side
* [ ] Test for JWT kid header injection (SQL injection, path traversal)

---

## Authorisation & Access Control

* [ ] Test all authenticated endpoints for unauthenticated access
* [ ] Test for Insecure Direct Object References (IDOR) on all resource identifiers
* [ ] Test horizontal privilege escalation (accessing another user's resources)
* [ ] Test vertical privilege escalation (accessing higher privilege functionality)
* [ ] Test for IDOR on numeric IDs, GUIDs, usernames, and email addresses
* [ ] Test for IDOR via HTTP method manipulation (GET → POST, PUT, DELETE)
* [ ] Test for IDOR on file download and upload endpoints
* [ ] Check if API endpoints enforce the same access controls as the UI
* [ ] Test for forced browsing to admin functionality
* [ ] Check for role or permission parameters that can be manipulated client-side
* [ ] Test multi-tenancy isolation — can one tenant access another's data?
* [ ] Test account switching / impersonation functionality for authorisation flaws

---

## Input Validation & Injection

### SQL Injection
* [ ] Test all user-supplied input for SQL injection (GET, POST, Headers, Cookies)
* [ ] Test for error-based SQL injection
* [ ] Test for blind boolean-based SQL injection
* [ ] Test for time-based blind SQL injection
* [ ] Test for second-order SQL injection
* [ ] Test for SQL injection in ORDER BY and LIMIT clauses
* [ ] Use SQLMap for automated confirmation and extraction

### Cross-Site Scripting (XSS)
* [ ] Test all reflected input fields for reflected XSS
* [ ] Test stored input fields for stored XSS (comments, profiles, names)
* [ ] Test for DOM-based XSS via client-side JavaScript source/sink analysis
* [ ] Test for XSS filter bypass via encoding, case variation, and tag confusion
* [ ] Check for XSS in HTTP headers rendered in responses (User-Agent, Referer)
* [ ] Test for XSS in file upload filenames and metadata
* [ ] Test for XSS in SVG file uploads
* [ ] Assess impact — can XSS be used to steal sessions, perform CSRF, or escalate?

### Command Injection
* [ ] Test for OS command injection in any functionality that interacts with the system (ping, traceroute, file conversion, DNS lookup features)
* [ ] Test blind command injection via time delays and out-of-band (OAST) techniques
* [ ] Test for argument injection where full command injection is not possible

### XML / XXE
* [ ] Test all XML input for XXE (file read, SSRF, blind OOB XXE)
* [ ] Test for XXE via file upload (SVG, DOCX, XLSX, PDF parsers)
* [ ] Test for XXE in SAML requests
* [ ] Test for XXE when JSON can be switched to XML via Content-Type manipulation

### Template Injection (SSTI)
* [ ] Test for SSTI in all reflected user input using polyglot payloads
* [ ] Identify template engine via error messages and engine-specific payloads
* [ ] Test for client-side template injection (AngularJS, Vue)

### Path Traversal
* [ ] Test for path traversal in file download, include, and upload functionality
* [ ] Test with encoding variations (URL encode, double encode, Unicode)
* [ ] Test on Windows targets with backslash variations
* [ ] Attempt to read sensitive files (/etc/passwd, web.config, application source)

---

## File Upload

* [ ] Test for unrestricted file upload (no extension validation)
* [ ] Test for MIME type bypass (change Content-Type to image/jpeg)
* [ ] Test for extension bypass (double extension, null byte, case variation)
* [ ] Test for web shell upload and execution
* [ ] Check upload destination — is it within the webroot?
* [ ] Test for path traversal in filename parameter
* [ ] Test for stored XSS via SVG upload
* [ ] Test for XXE via XML-based file format upload
* [ ] Check if uploaded files can be directly accessed by other users (IDOR)
* [ ] Test file size limits and assess DoS potential

---

## Cross-Site Request Forgery (CSRF)

* [ ] Check for CSRF token presence on all state-changing requests
* [ ] Test if CSRF token is validated server-side (remove or modify token)
* [ ] Test if CSRF token is tied to the user session
* [ ] Test for CSRF via GET request on state-changing actions
* [ ] Check SameSite cookie attribute — None/Lax/Strict implications
* [ ] Test for CSRF on logout functionality
* [ ] Test for CSRF bypass via content-type manipulation (JSON → form-data)

---

## Server-Side Request Forgery (SSRF)

* [ ] Identify all functionality that fetches remote URLs (webhooks, integrations, PDF generators, image fetchers)
* [ ] Test for SSRF to internal metadata service (169.254.169.254 for AWS/GCP/Azure)
* [ ] Test for SSRF to internal network hosts and services
* [ ] Test for blind SSRF via out-of-band (Burp Collaborator / interactsh)
* [ ] Test for SSRF bypass via DNS rebinding, IP encoding variations, and redirects
* [ ] Test for SSRF via file:// and dict:// protocol handlers
* [ ] Chain SSRF with internal service exploitation (Redis, Elasticsearch, internal APIs)

---

## Business Logic

* [ ] Map all application workflows and identify multi-step processes
* [ ] Test for ability to skip steps in multi-stage workflows
* [ ] Test for negative values in quantity, price, or transfer amount fields
* [ ] Test for race conditions on critical actions (payments, coupon redemption, transfers)
* [ ] Test for coupon/voucher reuse after single-use validation
* [ ] Test for price manipulation in purchase flows (modify hidden price fields)
* [ ] Test for mass assignment — can extra fields be submitted to elevate privileges?
* [ ] Test for account enumeration via business logic differences (checkout, invite flows)
* [ ] Test for feature access without entitlement (premium features on free accounts)

---

## API Testing

* [ ] Enumerate all API endpoints (Swagger/OpenAPI docs, JS file analysis, spider)
* [ ] Test for unauthenticated access to API endpoints
* [ ] Test all HTTP methods on each endpoint (GET, POST, PUT, PATCH, DELETE, OPTIONS)
* [ ] Test for mass assignment in API object creation and update endpoints
* [ ] Test for API versioning issues (v1 endpoint lacks controls enforced in v2)
* [ ] Test for excessive data exposure in API responses
* [ ] Test GraphQL for introspection enabled in production
* [ ] Test GraphQL for batch query abuse and field-level authorisation
* [ ] Test GraphQL for injection via query arguments
* [ ] Test for rate limiting on sensitive API endpoints (login, OTP, password reset)

---

## Cryptography & Transport Security

* [ ] Verify HTTPS is enforced across all endpoints (no HTTP fallback)
* [ ] Check TLS version and cipher suite strength (testssl.sh)
* [ ] Check for HSTS header presence and max-age value
* [ ] Check for mixed content issues (HTTP resources on HTTPS pages)
* [ ] Check for sensitive data transmitted in URL parameters (visible in logs/history)
* [ ] Verify sensitive data is not cached (Cache-Control, Pragma headers)
* [ ] Check for certificate pinning on mobile/thick clients

---

## Information Disclosure

* [ ] Review all error messages for stack traces, version info, or internal paths
* [ ] Test for verbose error messages by sending malformed input
* [ ] Check HTTP response headers for internal IP addresses or hostnames
* [ ] Check HTML source comments for credentials, internal references, or TODOs
* [ ] Check JavaScript files for hardcoded API keys, tokens, or credentials
* [ ] Test for directory listing on web directories
* [ ] Check for backup files (.bak, .old, .orig, ~) on known file paths
* [ ] Check for exposed admin interfaces (/admin, /manager, /phpmyadmin, /console)
* [ ] Test for source code disclosure (.php~, .php.bak, .git leakage)

---

## HTTP Headers & Security Misconfigurations

* [ ] Check for Content-Security-Policy (CSP) header and assess strictness
* [ ] Check for X-Frame-Options or CSP frame-ancestors (clickjacking)
* [ ] Check for X-Content-Type-Options: nosniff
* [ ] Check for Referrer-Policy header
* [ ] Check for Permissions-Policy header
* [ ] Test for CORS misconfiguration — does the server reflect arbitrary Origins?
* [ ] Test for CORS with null origin acceptance
* [ ] Test CORS with credentialed requests (withCredentials: true)
* [ ] Check OPTIONS responses for overly permissive CORS configuration
* [ ] Test for HTTP request smuggling (CL.TE, TE.CL) using Burp's HTTP Request Smuggler

---

## Rate Limiting & DoS

* [ ] Test for rate limiting on login, registration, and OTP endpoints
* [ ] Test for account lockout bypass via IP rotation or header manipulation (X-Forwarded-For)
* [ ] Test for large payload submission causing application errors
* [ ] Test for regex-based DoS (ReDoS) in input fields with complex validation
* [ ] Test for resource exhaustion via repeated expensive operations

---

## Post-Exploitation & Impact Assessment

* [ ] Document all findings with clear reproduction steps
* [ ] Assess CVSS score for each finding (use CVSSv4 where possible)
* [ ] Chain findings to demonstrate maximum impact (e.g. SSRF → IDOR → data exfiltration)
* [ ] Capture screenshots and HTTP request/response pairs for all findings
* [ ] Verify findings are reproducible before reporting
* [ ] Assess remediation complexity and provide actionable recommendations
