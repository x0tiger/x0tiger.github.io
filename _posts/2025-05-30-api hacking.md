---
published: true
title: API Hacking Part 1
date: 2025-05-30 02:40:00 +0200
categories: API-Hacking
tags:
  - API
  - Hacking
---
**The Foundations of API Hacking: A Practical Introduction**

---

### Part 1: Understanding APIs in the Real World

In today's digitally interconnected world, modern applications rarely operate in isolation. Instead, they interact constantly with other software, services, and platforms. At the core of this interaction is a concept known as the **API** — _Application Programming Interface_. Whether you're using a weather app, logging into a website using your Google account, or syncing your fitness watch with your phone, you're interacting with APIs every day.

---

### What Is an API?

An **API** is a set of rules and protocols that allows different software components to communicate with each other. It defines:

- **What** data or service can be requested
    
- **How** to make the request
    
- **What kind of response** to expect
    

**Real-world analogy:**  
Imagine a restaurant. The **menu** is the API. It tells you what you can order. When you order, the **waiter** (like the API) takes your request to the kitchen (server), and brings the result (response) back to you.

---

### Why Are APIs Important?

- **Separation of Concerns**: Frontend and backend can evolve independently.
    
- **Faster Development**: Reuse existing APIs instead of building from scratch.
    
- **Third-Party Integration**: Integrate payment, social media, maps, etc.
    
- **Scalability**: Services can scale separately when decoupled through APIs.
    

> **Security Note:** APIs often expose sensitive operations and data. A single misconfigured endpoint can lead to data leakage, unauthorized access, or full system compromise. That's why understanding APIs is essential for ethical hackers and penetration testers.

---

### Types of Web APIs

We mainly focus on **Web APIs**: APIs that operate over the internet using **HTTP/S** protocols. Key styles include:

#### 1. REST (Representational State Transfer)

- Uses HTTP verbs like `GET`, `POST`, `PUT`, `DELETE`
    
- Stateless communication
    
- Typically exchanges **JSON** data
    

**Example:**

```http
GET /users/123 HTTP/1.1
Host: api.example.com
```

**Response:**

```json
{
  "id": 123,
  "name": "elhussieny",
  "email": "elhussieny@example.com"
}
```

**Use cases:** Mobile apps, Microservices, Public APIs like Twitter

---

#### 2. SOAP (Simple Object Access Protocol)

- Uses **XML** messages
    
- Heavily used in legacy and enterprise environments
    
- Supports advanced standards (WS-Security)
    

**Example:**

```xml
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <m:GetUser xmlns:m="http://example.com/users">
      <m:UserID>123</m:UserID>
    </m:GetUser>
  </soap:Body>
</soap:Envelope>
```

**Use cases:** Banking, Government, ACID-compliant systems

---

#### 3. GraphQL

- Allows clients to request exactly what they need
    
- Single endpoint (usually `/graphql`)
    
- Can reduce over-fetching
    

**Example Query:**

```graphql
query {
  user(id: "user789") {
    id
    name
    orders {
      id
      status
      total
    }
  }
}
```

**Response:**

```json
{
  "data": {
    "user": {
      "id": "user789",
      "name": "John Doe",
      "orders": [
        {
          "id": "order456",
          "status": "Shipped",
          "total": 59.99
        }
      ]
    }
  }
}
```

---

### Authentication vs Authorization

- **Authentication**: Are you who you say you are?
    
- **Authorization**: Are you allowed to do what you're trying to do?
    

#### Common Authentication Methods

1. **Basic Auth**
    
    - Sends Base64 encoded username:password
        
    - Insecure unless used over HTTPS
        
    - ⚠️ Easily exploited if intercepted
        
2. **API Keys**
    
    - Static token used to identify client app
        
    - Often leaked in frontend code or GitHub repos
        
3. **Bearer Tokens / JWT**
    
    - Contains identity and claims in encoded format
        
    - Structure: `header.payload.signature`
        
    - ⚠️ Exploit: If server accepts `alg: none`, attacker can forge token
        
4. **OAuth 2.0**
    
    - Allows third-party access with user consent
        
    - Used for social logins and delegated access
        
5. **OpenID Connect**
    
    - Built on top of OAuth 2.0
        
    - Adds ID token to confirm identity of user
        

---

### Access Control Models

#### RBAC (Role-Based Access Control)

- Permissions are assigned based on user roles
    
- Example: Only Admins can delete users
    

#### ABAC (Attribute-Based Access Control)

- Access decisions are made based on attributes (location, time, department)
    
- Example: HR staff can access employee data only within their department
    

---

### API Architecture Styles

#### Monolithic

- Single codebase
    
- Easy to build, hard to scale
    

#### Microservices

- Split into smaller services connected by APIs
    
- Easier to scale and test
    

#### API Gateway

- Central point to route requests to microservices
    
- Manages caching, rate limits, and logging
    

#### Serverless / FaaS

- Code runs on-demand (e.g., AWS Lambda)
    
- Lower cost but harder to debug
    

#### GraphQL

- Query language for flexible data fetching
    
- Requires query validation and rate limits for security
    

---

### API Versioning

- Use `/v1/`, `/v2/`, or headers to version APIs
    
- Prevents breaking changes for existing clients
    

---

### Logging & Monitoring

Logs help detect:

- Abuse or brute-force attempts
    
- Malformed requests
    
- API usage trends
    

**Log Fields to Include:**

- Method and URL
    
- Status code
    
- Timestamp and IP address
    

⚠️ Never log passwords, tokens, or sensitive data

---

### API Documentation — A Developer's Lifeline

#### Why Documentation Matters

- Reduces onboarding time
    
- Improves usability and security
    
- Helps during audits and incident response
    

#### Good Documentation Includes:

1. **Endpoints & Methods**
    
2. **Parameters and Types**
    
3. **Example Requests & Responses**
    
4. **Authentication Methods**
    
5. **Error Codes and Meanings**
    

#### REST Example: Create Order

```http
POST /api/orders
Authorization: Bearer eyJhbGciOiJIUzI1...
Content-Type: application/json

{
  "productId": "abc123",
  "quantity": 2,
  "customerId": "user789"
}
```

**Success (201):**

```json
{
  "orderId": "order456",
  "status": "Order placed successfully"
}
```

**Error (401):**

```json
{
  "error": "Unauthorized"
}
```

---

#### SOAP Example: GetOrderStatus

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ord="http://example.com/orders">
  <soapenv:Body>
    <ord:GetOrderStatusRequest>
      <ord:OrderId>order456</ord:OrderId>
    </ord:GetOrderStatusRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

---

#### GraphQL Example:

```graphql
query {
  product(id: "abc123") {
    name
    price
  }
}
```

---

### Bonus: Example OpenAPI (Swagger) Spec

```yaml
openapi: 3.0.0
info:
  title: Example Orders API
  version: 1.0.0
paths:
  /api/orders:
    post:
      summary: Create a new order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Order'
      responses:
        '201':
          description: Order created
components:
  schemas:
    Order:
      type: object
      required:
        - productId
        - quantity
        - customerId
      properties:
        productId:
          type: string
        quantity:
          type: integer
        customerId:
          type: string
```

---

### Final Tips for API Security and Documentation

- Use HTTPS only
    
- Validate input and sanitize output
    
- Rate limit sensitive endpoints
    
- Regularly audit exposed endpoints
    
- Keep documentation updated and secure
    

---

### Resources & Tools

- [Postman](https://www.postman.com/)
    
- [Swagger Editor](https://editor.swagger.io/)
    
- [OAuth 2.0 Playground](https://developers.google.com/oauthplayground)
    

---

### Conclusion

APIs are the backbone of digital interaction. For developers and security professionals alike, understanding their structure, behavior, and vulnerabilities is a must. With great flexibility comes great responsibility — make your APIs secure, well-documented, and resilient.

---

> This guide serves as a practical foundation for learning, testing, and securing APIs in real-world environments.
