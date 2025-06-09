---
published: true
title: API Hacking Part 2
date: 2025-06-10 12:30:00 +0200
categories: API-Hacking
tags:
  - API
  - Hacking
---
# Part 2: Tools to Interact with an API

Once you understand the fundamentals of APIs—what they are, why they matter, and how they’re structured—the next logical step is learning how to effectively communicate with them. In this chapter, we explore a range of essential tools used to send requests and examine responses from APIs. These tools include command-line utilities, GUI-based testing platforms, and scripting languages. Whether you are debugging a REST endpoint, testing a SOAP service, or automating API calls, mastering these tools will greatly enhance your skills.

---

## 1. cURL: The Command-Line Workhorse

### 1.1 What Is cURL?

cURL (short for Client URL) is a command-line tool used to transfer data using various network protocols, including HTTP, HTTPS, FTP, and more. It is popular among developers, penetration testers, and DevOps engineers due to its simplicity, versatility, and ability to run in headless environments.

### 1.2 Why Use cURL?

- It allows fine control over HTTP methods, headers, and data payloads.
    
- It's lightweight and doesn't require a GUI.
    
- Excellent for quick testing, scripting, and automation.
    
- Works well in CI/CD pipelines.
    

### 1.3 Basic Examples

**GET request:**

```bash
curl https://api.example.com/users
```

**POST request with form data (application/x-www-form-urlencoded):**

```bash
curl -X POST -d "username=john&password=pass123" https://api.example.com/login
```

**POST request with JSON:**

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"username": "john", "password": "pass123"}' \
https://api.example.com/login
```

**Add custom headers (e.g., Authorization):**

```bash
curl -H "Authorization: Bearer <TOKEN>" https://api.example.com/protected
```

### 1.4 Authentication Methods

- **Basic Auth:**
    

```bash
curl -u username:password https://api.example.com/auth
```

- **Bearer Token:**
    

```bash
curl -H "Authorization: Bearer <TOKEN>" https://api.example.com/data
```

- **API Key:**
    

```bash
curl -H "x-api-key: <API_KEY>" https://api.example.com/data
```

### 1.5 Upload/Download Files

**Upload File:**

```bash
curl -F "file=@/path/to/file.txt" https://api.example.com/upload
```

**Download File:**

```bash
curl -o image.jpg https://api.example.com/file.jpg
```

### 1.6 Real-World Scenario

You are testing a login API during a penetration test. You use cURL to send different payloads and observe how the system behaves when you send invalid tokens, missing headers, or SQL payloads.

---

## 2. Postman: A GUI for API Exploration

### 2.1 What Is Postman?

Postman is a desktop and web-based application that provides a graphical interface to work with APIs. It is widely used by developers and QA teams for testing, documentation, and collaboration.

### 2.2 Why Use Postman?

- User-friendly interface—no need to memorize CLI syntax.
    
- Built-in tools for authentication, body formatting, and response inspection.
    
- Allows you to group and document requests.
    
- Supports environments (e.g., dev, staging, prod).
    
- Excellent for teamwork via collections and workspaces.
    

### 2.3 Making Requests in Postman

1. Open Postman and create a new request.
    
2. Choose the HTTP method (GET, POST, etc.).
    
3. Enter the endpoint URL.
    
4. Add headers, query parameters, or body as needed.
    
5. Click **Send** and review the response.
    

### 2.4 Collections & Environments

- **Collections:** Group API calls related to the same app/module.
    
- **Environments:** Set variables (like base URLs, tokens) to reuse across multiple requests.
    

### 2.5 Advanced Features

- **Authentication Support:** Basic Auth, Bearer Tokens, OAuth 2.0.
    
- **Tests and Scripts:** Write JavaScript to validate response content.
    
- **Newman CLI:** Run Postman collections via command-line for automation.
    
- **Mock Servers:** Simulate API responses when the real API isn't ready.
    

### 2.6 Real-World Scenario

A QA tester wants to test the entire login, profile view, and update process. They use Postman collections with test scripts to validate response codes and fields like `user_id` or `token`.

---

## 3. SOAP UI: Testing Tool for SOAP and REST

### 3.1 What Is SOAP UI?

SOAP UI is a desktop tool originally designed to test SOAP APIs, but it now supports REST APIs as well. It excels in structured test cases, assertions, and automation.

### 3.2 Features

- Import WSDL to auto-generate SOAP requests.
    
- Create and organize test suites and test cases.
    
- Data-driven tests (e.g., loop over CSV data).
    
- Supports Groovy scripting for custom logic.
    

### 3.3 SOAP Example

1. Import WSDL into SOAP UI.
    
2. Review the auto-generated requests.
    
3. Customize payload.
    
4. Send and validate the XML response.
    

### 3.4 REST Example

1. Create REST project.
    
2. Add endpoint, method, and parameters.
    
3. Create assertions to validate JSON/XML fields.
    

### 3.5 Real-World Scenario

A financial company uses SOAP APIs for transaction processing. The test team uses SOAP UI to validate XML structure, response codes, and to simulate transaction failure scenarios.

---

## 4. Python: Scripting & Automation for APIs

### 4.1 Why Use Python?

- Simple syntax.
    
- Rich ecosystem (e.g., `requests`, `httpx`).
    
- Good for automation and integrations.
    

### 4.2 Making Requests with `requests` Library

**Install:**

```bash
pip install requests
```

**GET Request:**

```python
import requests
response = requests.get("https://api.example.com/data")
print(response.status_code)
print(response.json())
```

**POST with form data:**

```python
data = {"username": "john", "password": "pass123"}
response = requests.post("https://api.example.com/login", data=data)
print(response.text)
```

**POST with JSON:**

```python
import json
headers = {"Content-Type": "application/json"}
data = {"key": "value"}
response = requests.post("https://api.example.com/json", headers=headers, json=data)
```

### 4.3 Headers and Auth

```python
headers = {
    "Authorization": "Bearer <TOKEN>",
    "Content-Type": "application/json"
}
response = requests.get("https://api.example.com/protected", headers=headers)
```

### 4.4 File Upload/Download

```python
# Upload
files = {"file": open("file.txt", "rb")}
response = requests.post("https://api.example.com/upload", files=files)

# Download
with open("output.jpg", "wb") as f:
    f.write(response.content)
```

### 4.5 Real-World Scenario

You're building a bot that automatically pulls product data every hour from an eCommerce API and saves it to a local database. You use Python scripts and schedule them with `cron` or Task Scheduler.

---

## Summary Table

|Tool|Type|Best For|
|---|---|---|
|cURL|CLI|Quick testing, automation, CI/CD|
|Postman|GUI|Prototyping, teamwork, manual testing|
|SOAP UI|GUI|Complex test cases, SOAP services|
|Python|Script|Automation, integrations, bots|

---

## Tips for Beginners

- Practice sending GET/POST requests to public APIs like [https://reqres.in](https://reqres.in) or [https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com)
    
- Watch tutorials for each tool on YouTube if anything is unclear.
    
- Use Postman’s built-in code generator to export cURL or Python snippets.
    

---

## Next Steps

- Learn how to integrate these tools in CI/CD pipelines.
    
- Explore security testing tools like OWASP ZAP and Burp Suite.
    
- Dive into API mocking, throttling, and advanced authentication flows (e.g., OAuth2 flows).
    

Mastering these tools ensures you're well-equipped to debug, test, and automate APIs across various platforms and environments.
