# 🌐 Lab 20: REST API Architecture — Mechanics, Authentication & Security Assessment

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-20-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

**REST (Representational State Transfer) APIs** form the primary communication architecture between modern web applications, mobile apps, and backend microservices over HTTP. Data is predominantly exchanged in structured **JSON** formats.

For penetration testers, security auditing of REST APIs focuses on verifying how servers enforce authorization boundaries, parse HTTP methods, discover hidden endpoints, and manage stateless authentication.

---

## ⚙️ Core Components: Endpoints & HTTP Methods

An **Endpoint** represents a specific resource path (URI), while the **HTTP Method** defines the requested action.

```
+-------------------+--------------------------------+--------------------------------------+
| HTTP Method       | Example Endpoint               | Action & Semantic Behavior           |
+-------------------+--------------------------------+--------------------------------------+
| GET               | /api/v1/users                  | Read / Retrieve resource list        |
| POST              | /api/v1/users                  | Create / Insert new resource         |
| PUT               | /api/v1/users/12               | Replace / Complete update of resource|
| PATCH             | /api/v1/users/12               | Partial update of resource fields    |
| DELETE            | /api/v1/users/12               | Remove / Delete resource             |
+-------------------+--------------------------------+--------------------------------------+
```

* **`GET`**: Read-only operation. Should not modify backend state.
* **`POST`**: Submits data to create a new resource.
* **`PUT`**: Completely overwrites an existing object with the provided body payload.
* **`PATCH`**: Modifies specified fields of an existing object without overwriting unreferenced attributes.
* **`DELETE`**: Permanently removes the target resource ID.

---

## 🔐 REST API Authentication Schemes

Because REST APIs are stateless, client requests must carry authentication tokens within HTTP Headers:

### 1. Bearer Token / JWT
The client transmits a signed token in the `Authorization` header:
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 2. API Key
The client passes a unique secret string via custom headers or query parameters:
```http
X-API-Key: secret_live_key_998877
```

### 3. Basic Authentication
Credentials are transmitted as Base64-encoded `username:password` strings:
```http
Authorization: Basic dXNlcjpwYXNz
```

> [!CAUTION]
> **Security Requirement:**  
> Base64 is an **encoding scheme**, not encryption. Basic Authentication must strictly operate over TLS/HTTPS to prevent cleartext credential harvesting.

---

## 🛠️ Pentesting Vectors & Attack Surface

```mermaid
flowchart TD
    A["👤 Authenticated Client (User 88)"] -->|"1. GET /api/v1/profiles/89"| B["🛡️ API Gateway / WAF"]
    
    B -->|"2. Checks Token Validity"| C{"Is Token Valid?"}
    
    C -->|"Yes"| D["⚙️ Backend API Engine"]
    C -->|"No"| E["🚫 401 Unauthorized"]
    
    D -->|"3. Evaluates Object Ownership (BOLA Check)"| F{"Validates Ownership<br/>User 88 == Object 89?"}
    
    F -->|"Flaw: Missing Object Control"| G["💥 200 OK + Returns User 89 Data (BOLA/IDOR)"]
    F -->|"Proper Control"| H["🚫 403 Forbidden"]

    classDef client fill:#f9f,stroke:#333,stroke-width:2px;
    classDef gateway fill:#ffe6cc,stroke:#d79b00,stroke-width:2px;
    classDef backend fill:#d5e8d4,stroke:#82b366,stroke-width:2px;
    classDef result fill:#f8cecc,stroke:#b85450,stroke-width:2px;

    class A client;
    class B,C gateway;
    class D,F backend;
    class E,G,H result;
```

### 1. Endpoint Discovery & Version Leakage
Exposing unlinked or legacy API endpoints (e.g., `/api/v1/`, `/api/v2/`, `/api/dev/`, `/api/test/`) can grant access to deprecated code paths that lack modern security filters.

### 2. Broken Object Level Authorization (BOLA / IDOR)
BOLA occurs when an API endpoint accepts object identifiers (`/api/v1/invoices/101`) without verifying whether the requesting user owns that object. It ranks as the **#1 threat in the OWASP API Security Top 10**.

### 3. HTTP Method Tampering
Security controls might block `GET` or `DELETE` requests while failing to restrict alternative verbs like `POST`, `PUT`, or `PATCH` targeting the same resource path.

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Field Scenario: Mobile App API Assessment

> [!CAUTION]
> **Field Condition:**  
> During an audit of a mobile application API, you intercept a request retrieving profile details:  
>
> `GET /api/v1/profiles/88`  
>
> **Server Response (`200 OK`):**
> ```json
> {
>   "id": 88,
>   "username": "ahmed",
>   "email": "ahmed@test.com"
> }
> ```

* **Questions:**
  1. What vulnerability are you immediately testing for when changing the URI parameter from `88` to `89`?
  2. Which standard HTTP method should be used if you want to submit a request to update *only* the `email` field while leaving all other user fields untouched?

---

* **Student Analysis:**
  > **1. Target Vulnerability:** Changing the ID to `89` directly tests for **Insecure Direct Object References (IDOR)** / **Broken Object Level Authorization (BOLA)** to determine if User `88` can access User `89`'s profile data.  
  > **2. HTTP Method Choice:** The **`PATCH`** method is the RFC-standard verb for partial updates.

---

* **Technical Security Breakdown & Evaluation:**
  * **Verdict:** ✅ **100% Accurate Assessment.**
  * **BOLA / IDOR Evaluation (Question 1):**  
    Substituting `88` with `89` tests whether the API enforces **Object-Level Authorization**. If the server returns User `89`'s private JSON object without validating token ownership, a critical BOLA vulnerability is confirmed.
  * **HTTP Method Mechanics (Question 2):**  
    * `PATCH /api/v1/profiles/88` with body `{"email": "new@test.com"}` updates only the specified field.
    * Using `PUT` would expect a complete entity representation, potentially clearing unsupplied fields (e.g., `username`) depending on backend implementation.

---
