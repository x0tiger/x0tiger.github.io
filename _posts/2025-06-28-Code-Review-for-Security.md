---
published: true
title: Secure Code Review
date: 2025-06-28 10:30:00 +0200
categories: Code-Review
tags:
  - Source
  - Review
---

---

# Secure Code Review

## 1. Introduction

Secure code review is the cornerstone of modern application security. It’s not just about spotting `eval()` calls or SQL queries—it's about deeply understanding how the application behaves, how logic is constructed, and where assumptions break. In real-world, large-scale systems, the code is modular, distributed, and often event-driven. Security auditors must go beyond syntax to assess **business logic flaws**, **authentication boundaries**, and **data flow risks**.

---

## 2. What is Secure Code Review?

Secure code review is the process of analyzing application source code to find security vulnerabilities—either manually, using automated tools, or both. While static analysis tools can catch known patterns, **manual inspection** is essential to catch context-specific issues like **access control violations**, **broken business logic**, and **dangerous assumptions**.

>  “Tools can find bugs. Humans find flaws.”

---

## 3. Mapping the Codebase: Understand Before You Review

Before diving into the code, answer the following:

|Key Question|Purpose|
|---|---|
|How is the code structured?|Identify modules and responsibilities|
|Where does user input enter?|Entry points = attack surface|
|How is data processed and stored?|Trace potential data exposure|
|What is the control flow style?|Event-driven? REST? Queue-based?|

###  Example Project Layout (Node.js)

```bash
src/
├── controllers/
├── services/
├── routes/
├── middlewares/
├── models/
├── config/
└── utils/
```

 **Tip:** Scan the routes and entry points first to determine where sensitive logic resides.

---

## 4. Entry Points and Trust Boundaries

Your code review should prioritize "trust boundaries"—the points where **external input enters the system**.

### Common Trust Boundaries:

- HTTP request parameters (`req.body`, `req.query`)
    
- WebSocket messages
    
- API tokens and cookies
    
- File uploads
    
- Environment configurations
    

### Example: API Endpoint Vulnerability

```js
// ⚠️ Vulnerable: Missing authorization check
app.get('/api/users/:id', async (req, res) => {
  const user = await db.getUserById(req.params.id);
  res.json(user);
});
```

####  Secure Version

```js
app.get('/api/users/:id', authMiddleware, async (req, res) => {
  if (req.user.id !== req.params.id) {
    return res.status(403).json({ error: 'Unauthorized access' });
  }
  const user = await db.getUserById(req.params.id);
  res.json(user);
});
```

---

## 5. Understand the Programming Paradigm

Real-world systems are often:

- **Object-Oriented (OOP)**: Organize behavior into encapsulated units
    
- **Event-Driven**: Especially in Node.js, React, or backend message queues
    
- **Asynchronous**: Using promises, `async/await`, callbacks
    

### Implication for Reviewers:

- No central "start" function
    
- Need to trace _event handlers_ and _callbacks_
    
- Vulnerabilities can be hidden in edge-case handlers or race conditions
    

---

## 6. Business Logic Flaws: The Real Killers

Automated tools don’t catch these.

### Example: Broken Discount Logic

```js
if (coupon.applied && cart.total > 1000) {
  discount = coupon.amount;
}
```

####  Red Flags:

- Who can apply the coupon?
    
- Can the user control both `coupon` and `cart.total`?
    
- Is coupon reuse checked?
    
- What if total is manipulated via request tampering?
    

---

## 7. Authentication and Session Logic

### Example: Weak JWT Handling

```js
const token = jwt.sign({ id: user.id }, 'secretkey');
```

#### Problems:

- Hardcoded secret
    
- No expiry (`exp` claim)
    
- No issuer/audience validation
    

####  Improved Version:

```js
const token = jwt.sign(
  { id: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '1h', issuer: 'myApp' }
);
```

> 🧠 Always validate the JWT server-side even if the client sends it. Don’t assume `role` from the token without re-checking permissions.

---

## 8. Handling Input: The Gateway to Exploits

###  Dangerous Pattern:

```js
const user = await db.query(
  `SELECT * FROM users WHERE email = '${req.body.email}'`
);
```

###  Safer Version with Prepared Statements:

```js
const user = await db.query(
  `SELECT * FROM users WHERE email = $1`,
  [req.body.email]
);
```

> ✍️ Always sanitize and validate user input. Libraries like Joi (JS), Cerberus (Python), or Zod (TS) help enforce schema validation.

---

## 9. Race Conditions and Asynchronous Flaws

### Vulnerable Transfer Logic (Node.js)

```js
socket.on('transferFunds', async ({ from, to, amount }) => {
  await withdraw(from, amount);
  await deposit(to, amount);
});
```

####  Exploit Vector:

If called in quick succession, race conditions might allow over-withdrawal or double-deposit.

####  Mitigation:

- Use **atomic database operations** or transactions
    
- Lock resources temporarily
    
- Rate-limit socket events
    

---

## 10. Reviewing Test Code for Security Gaps

Security-relevant tests:

- Input validation edge cases
    
- Authentication bypass attempts
    
- Privilege escalation tests
    
- Denial-of-service simulations
    
- Invalid token handling
    

 Use **mutation testing** tools (e.g., Stryker, Mutant) to assess whether your tests can detect broken or altered logic.

---

## 11. Advanced Red Flags to Look For

|Risk|Code Pattern|
|---|---|
|Arbitrary file read|`fs.readFile(req.query.path)`|
|Unsafe eval|`eval(req.body.script)`|
|Insecure redirects|`res.redirect(req.query.url)`|
|SSRF|`axios.get(req.body.url)`|
|No rate limit|Missing IP-throttling middleware|
|Sensitive keys in repo|`.env`, `API_SECRET`, `config.js`|

---

## 12. Automation + Manual = Balance

### 🔧 Recommended Tooling

|Tool|Purpose|
|---|---|
|**Semgrep**|Rule-based static analysis|
|**CodeQL**|Code query language for deep queries|
|**SonarQube / SonarLint**|Quality + security smell detection|
|**FindSecBugs (Java)**|Security plugin for SpotBugs|
|**Bandit (Python)**|Static analyzer for Python code|
|**AST Viewers**|Analyze language-specific trees for patterns|

---

## 13. Tips for Reviewing Large Codebases

- **Don’t read everything.** Focus on modules relevant to the audit goal (e.g., auth, payments).
    
- **Use folder conventions** to identify sensitive code (e.g., `auth/`, `admin/`, `handlers/`).
    
- **Trace feature flow**, not file order: Follow the flow from UI -> Controller -> Service -> DB.
    
- **Identify dangerous patterns** like global state mutations, hardcoded secrets, or bypassable guards.
    
- **Search for bad assumptions**, not just bad syntax.
    

---

## 14. Conclusion: Think Like an Architect, Review Like an Attacker

Secure code review is both **strategic and surgical**. It requires:

- Architectural understanding
    
- Practical attacker mindset
    
- Familiarity with frameworks and language quirks
    
- Deep knowledge of the OWASP Top 10, CWE, and custom logic flaws
    

> “The goal isn’t to find one bug. The goal is to understand how bugs emerge from assumptions.”

---
