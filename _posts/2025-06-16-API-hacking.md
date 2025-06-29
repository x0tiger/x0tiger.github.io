---
published: true
title: OWASP API Top 10
date: 2025-06-16 10:30:00 +0200
categories: API-Hacking
tags:
  - API
  - Hacking
---
# OWASP API Top 10 - Combined (2019 & 2023)

APIs are the backbone of modern applications. Any vulnerability in them can lead to a full system compromise. This combined guide merges the OWASP API Top 10 lists from 2019 and 2023, integrating both lists into a unified, practical reference. Each issue includes descriptions , root causes, real-world attack scenarios, exploitation examples, and mitigation strategies.

---

## 1. Broken Object Level Authorization (BOLA)

**Description:**  
Occurs when the API fails to properly enforce authorization checks on object-level access. Attackers manipulate object IDs (like user_id or order_id) to access other users' data.

**Why it happens:**  
The server uses object IDs in URLs or body parameters but doesn’t verify object ownership.

**Exploitation Example:**

```http
GET /api/appointments/246
Authorization: Bearer user-token
```

**Realistic Scenario:**  
A researcher changes the appointment ID in a healthcare app and accesses another patient’s data.

**Mitigation:**

- Enforce authorization on the server side for every request
    
- Use non-predictable IDs (e.g., UUIDs)
    
- Centralize and standardize access control
    

---

## 2. Broken User Authentication

**Description:**  
Weak or misconfigured authentication allows attackers to gain unauthorized access to user accounts.

**Why it happens:**

- Poor password policies
    
- Token reuse
    
- Lack of rate-limiting
    
- Broken session handling
    

**Exploitation Example (Brute-force):**

```bash
for pass in $(cat passwords.txt); do
  curl -X POST https://api.site.com/login \
  -d '{"email":"victim@example.com", "password":"'"$pass"'"}'
done
```

**Realistic Scenario:**  
An attacker reuses an old password reset link to gain access to other users' accounts.

**Mitigation:**

- Invalidate tokens on logout
    
- Rate-limit login attempts
    
- Use MFA and secure password reset flows
    

---

## 3. Broken Object Property Level Authorization / Mass Assignment

**Description:**  
APIs auto-bind user input to backend objects. Attackers can include unauthorized fields (e.g., `role`, `isAdmin`) in requests.

**Why it happens:**  
Automatic data binding without filtering allows users to modify sensitive object properties.

**Exploitation Example:**

```json
POST /api/users/update
{
  "username": "ali",
  "isAdmin": true
}
```

**Realistic Scenario:**  
An attacker adds `"isAdmin": true` during registration and gains admin privileges.

**Mitigation:**

- Use strict input validation and DTOs
    
- Whitelist allowed properties
    
- Reject unknown/sensitive fields
    

---

## 4. Excessive Data Exposure

**Description:**  
APIs return more data than necessary, exposing sensitive details that aren’t needed by the client.

**Why it happens:**  
Developers rely on the front-end to filter data instead of controlling output at the API level.

**Exploitation:**

- Analyze API responses for hidden fields using tools like Burp Suite
    

**Realistic Scenario:**  
A banking app's API returns account numbers and national IDs, even though the front-end only displays the name and balance.

**Mitigation:**

- Review API responses during development
    
- Use Data Transfer Objects (DTOs)
    
- Return only necessary data
    

---

## 5. Lack of Resource & Rate Limiting / Unrestricted Resource Consumption

**Description:**  
APIs without limits on the number, size, or frequency of requests can be abused to cause Denial-of-Service (DoS) or brute-force attacks.

**Why it happens:**

- No rate limiting or payload size restrictions
    
- APIs allow repeated, costly operations
    

**Exploitation Example:**

```bash
curl -X POST https://api.site.com/upload \
  -d @large_file.json
```

**Realistic Scenario:**  
A food delivery app is flooded with thousands of fake orders, impacting a competitor’s operations.

**Mitigation:**

- Apply rate limits (e.g., API gateway like Kong)
    
- Restrict request size
    
- Use CAPTCHA or challenge-response for repeated access
    

---

## 6. Broken Function Level Authorization

**Description:**  
APIs expose functions (e.g., admin endpoints) that are not properly protected by user roles or permissions.

**Why it happens:**  
Developers rely on frontend restrictions and forget server-side role checks.

**Exploitation Example:**

```http
POST /api/admin/deleteUser
Authorization: Bearer user-token
```

**Realistic Scenario:**  
A regular user calls an admin-only endpoint and deletes tickets or users.

**Mitigation:**

- Implement Role-Based or Attribute-Based Access Control (RBAC/ABAC)
    
- Enforce authorization at all layers
    

---

## 7. Unrestricted Access to Sensitive Business Flows

**Description:**  
Some business-critical operations like bulk deletion or data export are exposed without sufficient controls.

**Why it happens:**  
Developers don’t apply additional verification because such endpoints are rarely used by standard users.

**Exploitation Example:**

```json
POST /api/bulkDelete
{
  "userIDs": [101, 102, 103]
}
```

**Realistic Scenario:**  
An attacker finds and abuses an endpoint to mass delete accounts.

**Mitigation:**

- Mark high-impact endpoints for extra verification (e.g., MFA)
    
- Monitor logs and set rate limits
    

---

## 8. Server Side Request Forgery (SSRF)

**Description:**  
Occurs when APIs fetch URLs from user input without validating the target. This allows attackers to reach internal services.

**Why it happens:**  
No filtering or restriction on user-supplied URLs.

**Exploitation Example:**

```json
POST /api/fetchUrl
{
  "url": "http://localhost:8080/admin"
}
```

**Realistic Scenario:**  
An attacker fetches internal admin panels or cloud metadata via open SSRF.

**Mitigation:**

- Whitelist allowed destinations
    
- Block internal IP ranges
    
- Limit server egress access
    

---

## 9. Security Misconfiguration

**Description:**  
APIs or services are deployed with insecure default settings, verbose errors, or exposed internal components.

**Why it happens:**  
Poor configuration practices, unused features enabled, or lack of hardening.

**Exploitation:**

- Access debug endpoints
    
- Scan for default credentials or open ports
    

**Realistic Scenario:**  
Redis exposed publicly without a password allows manipulation of sessions.

**Mitigation:**

- Disable unnecessary features
    
- Secure all environments (dev/stage/prod)
    
- Use automated configuration auditing
    

---

## 10. Improper Assets Management

**Description:**  
Old, undocumented, or staging APIs remain exposed in production.

**Why it happens:**  
Lack of API lifecycle management or version control.

**Exploitation:**

- Discover legacy endpoints using tools like FFUF or Dirsearch
    

**Realistic Scenario:**  
An attacker accesses `/api/v1/` which is unauthenticated and contains critical vulnerabilities.

**Mitigation:**

- Track all APIs
    
- Remove outdated versions
    
- Protect non-production endpoints
    

---

## 11. Injection (SQL/NoSQL/Command/LDAP)

**Description:**  
Occurs when untrusted input is interpreted as code, leading to data theft or system compromise.

**Why it happens:**  
Lack of input validation or unsafe query construction.

**Exploitation Example:**

```http
GET /api/products?sort=price); DROP TABLE users;--
```

**Realistic Scenario:**  
An e-commerce API allows command injection through query parameters.

**Mitigation:**

- Use parameterized queries
    
- Sanitize and validate all input
    
- Avoid dynamic code execution
    

---

## 12. Insufficient Logging & Monitoring

**Description:**  
APIs that lack proper logging, alerting, or audit trails allow attacks to go undetected.

**Why it happens:**  
Lack of centralized logging, no alert thresholds, or too many ignored events.

**Exploitation:**

- Simulate password reset abuse
    
- Trigger multiple errors and observe if alerts are fired
    

**Realistic Scenario:**  
Thousands of malicious password reset requests go unnoticed due to missing logs.

**Mitigation:**

- Log authentication, privilege changes, and sensitive API access
    
- Use SIEM solutions (e.g., Wazuh, Splunk)
    
- Regularly audit and alert on suspicious behavior
    

---

## Conclusion

Every vulnerability in the OWASP API Top 10 reflects a real-world risk that has been exploited repeatedly. Combining both the 2019 and 2023 lists provides a broader understanding of how APIs are attacked and how they should be defended. Secure design, continuous monitoring, and strict validation are non-negotiable for any API-first application.