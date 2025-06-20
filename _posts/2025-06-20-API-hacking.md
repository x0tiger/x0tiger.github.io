---
published: true
title: API Hacking Part 3
date: 2025-06-20 10:30:00 +0200
categories: API-Hacking
tags:
  - API
  - Hacking
---
### Part 3: OWASP API Top 10 - 2019 

APIs are the backbone of modern applications. Any vulnerability in them can lead to a full system compromise. In 2019, OWASP released its list of the top 10 most critical API security issues. Here’s an in-depth explanation of each, including very realistic attack scenarios and mitigation strategies.

---

#### 1. **Broken Object Level Authorization (BOLA)**

**Description:** Happens when the API does not properly validate whether the user is authorized to access a specific object like a user profile or an order.

**Highly Realistic Scenario:**

- A healthcare appointment app allows this request:
    
    ```
    GET /api/appointments/245
    ```
    
    A security researcher changes it to `/api/appointments/246` and gains access to another patient’s data without any access control.
    

**Exploitation:**

- Modify IDs like `user_id`, `order_id`, `appointment_id` in the URL or request body.
    

**Mitigation:**

- Always implement server-side authorization checks.
    
- Use non-guessable IDs like UUIDs.
    
- Apply a unified and consistent access control mechanism.
    

---

#### 2. **Broken User Authentication**

**Description:** Occurs when authentication mechanisms fail, allowing attackers to log in without valid credentials.

**Highly Realistic Scenario:**

- An online learning platform allowed password reset links to be reused indefinitely. An attacker reused an old link to reset other users' passwords.
    

**Exploitation:**

- Stolen JWTs or cookies.
    
- Brute-force login attempts.
    
- Poor session management.
    

**Mitigation:**

- Invalidate tokens on logout.
    
- Enforce MFA on sensitive operations.
    
- Rate-limit failed login attempts and alert on brute-force patterns.
    

---

#### 3. **Excessive Data Exposure**

**Description:** The API returns more data than necessary, exposing sensitive details even if they’re not shown in the front-end.

**Highly Realistic Scenario:**

- A banking API returns full user details in JSON, including account numbers, national ID, and internal status fields, though only name and balance are shown in the UI.
    

**Exploitation:**

- Analyze API responses using tools like Burp Suite to discover hidden fields.
    

**Mitigation:**

- Use DTOs to control returned fields.
    
- Review API responses during development and production.
    

---

#### 4. **Lack of Resource & Rate Limiting**

**Description:** The API lacks restrictions on the number or size of requests, allowing attackers to overload it or brute force credentials.

**Highly Realistic Scenario:**

- A food delivery app had no rate limits on the order API. An attacker flooded the system with fake orders to a competitor’s restaurant, rendering it unusable.
    

**Exploitation:**

- Send large payloads.
    
- Rapid, repeated requests.
    
- Brute-force token or login attempts.
    

**Mitigation:**

- Implement rate limits via API gateway (e.g., Kong).
    
- Set request size limits.
    
- Apply CAPTCHA or MFA after repeated access.
    

---

#### 5. **Broken Function Level Authorization**

**Description:** Attackers access privileged functions (e.g., admin features) without proper authorization checks.

**Highly Realistic Scenario:**

- An event ticketing app used similar endpoints for users and admins. A regular user discovered `POST /admin/delete_ticket` worked for them too.
    

**Exploitation:**

- Try invoking admin or restricted functions using low-privilege accounts.
    

**Mitigation:**

- Enforce Role-Based or Attribute-Based Access Control (RBAC/ABAC).
    
- Apply authorization checks at the gateway and within backend code.
    

---

#### 6. **Mass Assignment**

**Description:** When APIs automatically bind request data to backend models without filtering, attackers can assign sensitive fields.

**Highly Realistic Scenario:**

- During user registration, an attacker adds `"isAdmin": true` in the JSON body. The server accepts it and creates an admin account.
    

**Exploitation:**

- Inject hidden or unauthorized fields into the JSON payload.
    

**Mitigation:**

- Whitelist permitted fields.
    
- Use strict DTOs for input validation.
    
- Reject unknown or sensitive attributes.
    

---

#### 7. **Security Misconfiguration**

**Description:** Arises from default, incomplete, or insecure configurations in servers or components.

**Highly Realistic Scenario:**

- A warehouse system exposed Redis on a public IP without a password. An attacker accessed and manipulated session data.
    

**Exploitation:**

- Scan for open ports, default credentials, unnecessary features.
    

**Mitigation:**

- Disable unnecessary services.
    
- Change default passwords.
    
- Hide verbose error messages in production.
    

---

#### 8. **Injection (SQL/NoSQL/Command/LDAP)**

**Description:** Occurs when untrusted input is executed as a command or query.

**Highly Realistic Scenario:**

- An online store’s API had a filter parameter like `sort=price`. An attacker tried `sort=price); DROP TABLE users;--` and succeeded in deleting user data.
    

**Exploitation:**

- Test inputs with payloads like `' OR 1=1 --` or `sleep(5)`.
    

**Mitigation:**

- Use parameterized queries.
    
- Sanitize user input.
    
- Avoid dynamic query composition.
    

---

#### 9. **Improper Assets Management**

**Description:** Occurs when outdated or undocumented APIs are accessible in production.

**Highly Realistic Scenario:**

- A medical app launched `/api/v2`, but left `/api/v1` exposed. The old version lacked authentication and allowed full data access.
    

**Exploitation:**

- Use tools like FFUF or Dirsearch to discover legacy APIs.
    

**Mitigation:**

- Track all deployed APIs and environments.
    
- Decommission old APIs.
    
- Secure staging and dev endpoints.
    

---

#### 10. **Insufficient Logging & Monitoring**

**Description:** Without proper logging and alerting, malicious activities may go unnoticed.

**Highly Realistic Scenario:**

- An attacker tried thousands of password reset requests without triggering any alerts or being logged.
    

**Exploitation:**

- Simulate suspicious behavior and check for logs or alerts.
    

**Mitigation:**

- Log all critical actions: login attempts, privilege changes, admin actions.
    
- Integrate with SIEM tools (Splunk, Wazuh).
    
- Regularly review and audit logs.
    

---

### Conclusion:

Each item in the OWASP API Top 10 reflects a real-world weakness frequently exploited by attackers. A deep understanding of these issues—paired with strong testing and secure coding practices—is essential to protect APIs from abuse. Modern applications must treat security as a foundational element, not an afterthought..