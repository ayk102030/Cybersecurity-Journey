# 🌐 Lab 01: HTTP Methods & Cybersecurity Implications



## 📌 Overview

In the **HTTP** protocol, **Methods** (or verbs) specify the desired action to be performed on a given resource by the server. This lab covers the core mechanics of these methods, associated security risks, and their practical applications in penetration testing and web security.

---

## 📊 Quick Reference Table

| Method | Request Body? | Primary Purpose | Key Cybersecurity Risk |
| :--- | :---: | :--- | :--- |
| **`GET`** | ❌ No | Retrieve / Fetch Data | Information Disclosure (URL Leakage) |
| **`POST`** | ✅ Yes | Submit & Process Data | SQLi, XSS, CSRF |
| **`PUT`** | ✅ Yes | Replace / Upload Complete File | WebShell Upload, File Overwrite |
| **`PATCH`** | ✅ Yes | Partial Data Modification | Mass Assignment Vulnerability |
| **`DELETE`** | ❌ No | Remove Specified Resource | IDOR / Broken Access Control |
| **`OPTIONS`** | ❌ No | Discover Supported Methods | Reconnaissance & CORS Preflight |
| **`HEAD`** | ❌ No | Fetch Headers Only | Fast Recon & File Discovery |

---

## 🛠️ Detailed Technical Breakdown

### 1. `GET` (Retrieve Data)
Used exclusively to request data from a specified resource. Parameters are sent directly within the URL query string (e.g., `?id=5`).

> [!WARNING]
> **Information Disclosure:** Sensitive data (passwords, API keys, session tokens) must **never** be transmitted via `GET` requests. URLs are routinely logged in `Server Access Logs`, stored in `Browser History`, and leaked via the `Referer` header during outbound navigation.

---

### 2. `POST` (Submit & Process Data)
Used to send data to the server to create or process a resource. Data is packaged securely inside the **Request Body** rather than the URL.

> [!NOTE]
> `POST` endpoints are primary targets for **SQL Injection** and **Cross-Site Scripting (XSS)** because they process user input. They are also vulnerable to **CSRF (Cross-Site Request Forgery)** if anti-CSRF tokens are not enforced.

---

### 3. `PUT` vs `PATCH` (Modify & Update)
* **`PUT`:** Completely replaces a target resource or uploads a new file.
* **`PATCH`:** Applies partial modifications to a resource (e.g., updating a single user field).

> [!CAUTION]
> * **Mass Assignment:** Without strict server-side validation, an attacker can append unauthorized parameters to a `PATCH` request (e.g., `"is_admin": true`) to escalate privileges.
> * **WebShell Upload:** Unrestricted `PUT` implementations allow attackers to upload arbitrary malicious files directly (e.g., `webshell.php`).

---

### 4. `DELETE` (Remove Resource)
Requests the permanent deletion of a specified resource:

```http
DELETE /api/users/10 HTTP/1.1
Host: target.com
```

> [!IMPORTANT]
> The risk stems not from the method itself, but from missing authorization checks (**Broken Access Control / IDOR**). If a low-privileged user can delete another user's account by tampering with the ID, it represents a high-severity flaw.

---

### 5. `OPTIONS` (Reconnaissance & CORS)
Used to discover which HTTP methods are supported by a specific resource on the target server.

**Example Server Response:**
```http
HTTP/1.1 200 OK
Allow: GET, POST, PUT, DELETE, OPTIONS
```

> [!TIP]
> **Cybersecurity Perspective:**
> 1. **Reconnaissance:** Penetration testers use `OPTIONS` to enumerate enabled methods. *(Note: The presence of `PUT` or `DELETE` in the `Allow` header indicates support, but does not guarantee unauthenticated exploitation).*
> 2. **CORS Preflight:** Essential for Cross-Origin Resource Sharing (CORS). Browsers automatically send a preflight `OPTIONS` request before executing non-simple requests (like `PUT` or `DELETE` with custom headers) to verify server permissions.

---

### 6. `HEAD` (Headers Only)
Identical to `GET`, but the server returns only the **Response Headers** without the **Response Body**.

```http
HEAD /backup.zip HTTP/1.1
Host: target.com
```

**Server Response:**
```http
HTTP/1.1 200 OK
Content-Length: 152000000
Content-Type: application/zip
```

> [!NOTE]
> **Cybersecurity Perspective:**
> * Ideal for rapid asset discovery, verifying file existence, checking file sizes, and determining content types without consuming bandwidth.
> * **Correction / Clarification:** Using `HEAD` does **not** make requests invisible to Intrusion Detection Systems (`IDS/IPS`) or server logs. `HEAD` requests generate access log entries identically to `GET` requests and can trigger security alerts during broad scanning patterns.

---

## 🧪 Practical Scenarios & Analysis

<details>
<summary><b>🔹 Scenario 1: Password Change via GET Parameter</b></summary>

<br>

* **Condition:** A password reset endpoint passes parameters via URL:  
  `[https://target.com/change?pass=New123](https://target.com/change?pass=New123)`
* **Vulnerability Class:** Sensitive Data Exposure in URL / Information Disclosure.
* **Risk:** The new password (`New123`) is stored in plaintext across `Browser History`, `Proxy Logs`, `Server Access Logs`, and HTTP `Referer` headers.
* **Remediation:** Convert the endpoint to `POST`, pass parameters in the request body, and enforce HTTPS encryption.
</details>

<details>
<summary><b>🔹 Scenario 2: Detecting Unrestricted File Uploads & Modifications</b></summary>

<br>

* **Objective:** Identify methods that allow file modifications or arbitrary file uploads without proper authentication.
* **Assessment Methodology:**
  1. Issue an `OPTIONS` request to inspect the `Allow` response header.
  2. If `PUT` or `PATCH` are exposed, test file creation/modification payloads to verify if access control checks are actively enforced.
</details>

<details>
<summary><b>🔹 Scenario 3: Fast File Discovery & Enumeration</b></summary>

<br>

* **Objective:** Scan a target server for 10,000 hidden files with minimal bandwidth usage and maximum speed.
* **Selected Method:** **`HEAD`**
* **Rationale:** Using `GET` downloads the entire response body for every valid hit, causing severe network latency and bandwidth overhead. `HEAD` evaluates existence strictly through HTTP status codes (`200 OK` vs `404 Not Found`) without retrieving payload data.
</details>
