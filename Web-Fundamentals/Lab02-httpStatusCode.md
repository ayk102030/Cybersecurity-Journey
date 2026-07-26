# 🌐 Lab 02: HTTP Status Codes & Pentesting Implications



## 📌 Overview

In web security and penetration testing, **HTTP Status Codes** are critical signals returned by the server indicating how a request was processed. Understanding these codes allows penetration testers to identify hidden resources, analyze server behavior under unexpected input, bypass access restrictions, and uncover high-severity vulnerabilities like **SQL Injection**.

---

## 📊 Quick Reference Table

| Status Code | Description | Pentesting Significance | Primary Action / Vulnerability |
| :--- | :--- | :--- | :--- |
| **`200 OK`** | Success | Confirms resource existence | Directory Brute-Forcing target |
| **`301 / 302`** | Permanent / Temporary Redirect | Page location changed | Redirect analysis & Auth Bypass |
| **`401 Unauthorized`** | Authentication Required | User identity is unknown | Credential Brute-forcing |
| **`403 Forbidden`** | Access Denied | User identified, but permission denied | 403 Bypass (Headers / Path Traversal) |
| **`404 Not Found`** | Resource Missing | Endpoint does not exist | Ignored by tools (Watch out for Soft 404) |
| **`500 Internal Error`**| Unhandled Server Exception | Backend crash due to input | High indicator of **SQL Injection** |
| **`502 Bad Gateway`** | Upstream Server Failure | Proxy cannot reach backend | Potential Denial of Service (DoS) |

---

## 🛠️ Detailed Technical Breakdown

### 1. `200 OK` (Successful Response)
Indicates that the request succeeded and the server returned the requested resource.

> [!TIP]
> **Cybersecurity Perspective:**
> `200 OK` is the primary target for automated enumeration tools during directory brute-forcing (e.g., `Gobuster`, `ffuf`, `Dirsearch`). Discovering hidden endpoints returning `200 OK` (such as `/admin`, `.env`, or `/backup.zip`) often leads to sensitive information disclosure or administrative exposure.

---

### 2. `301 Moved Permanently` & `302 Found` (Redirections)
Indicates that the requested URI has been moved either permanently (`301`) or temporarily (`302`) to a new URL specified in the `Location` header.

> [!NOTE]
> **Cybersecurity Perspective:**
> Analysts monitor redirections closely. For example, if accessing an administrative panel redirects an unauthenticated user to `/login`, attackers test logic flaws or parameter manipulation to bypass the redirect and access the underlying page content directly.

---

### 3. `401 Unauthorized` vs `403 Forbidden` (Access Control)
Understanding the distinction between these two status codes is fundamental to web application testing:

* **`401 Unauthorized`:** The server does not know who you are. You must authenticate first (missing or invalid credentials).
* **`403 Forbidden`:** The server knows who you are, but you lack sufficient permissions to access the resource.

> [!CAUTION]
> **403 Bypass Techniques:**
> A `403 Forbidden` status confirms that the requested resource **exists**. Attackers frequently attempt 403 bypass techniques, such as:
> 1. **Header Tampering:** Injecting headers like `X-Forwarded-For: 127.0.0.1` or `X-Original-URL`.
> 2. **Path Manipulation:** Appending special characters or path traversal sequences (e.g., `/admin/.`, `/admin/..;/`, `/admin%20`).

---

### 4. `404 Not Found` & The "Soft 404" Threat
Indicates that the server cannot find the requested URL. Fuzzing tools automatically filter out `404` responses to clean up scan outputs.

> [!WARNING]
> **Soft 404 Hazard:**
> Some misconfigured servers respond with an HTTP `200 OK` status code while rendering a custom webpage that says *"Page Not Found"*. This is known as a **Soft 404** and can trick automated directory scanners into reporting false positives for non-existent files.

---

### 5. `500 Internal Server Error` (The PenTester's Best Friend)
Occurs when the server encounters an unexpected condition that prevents it from fulfilling the request.

> [!IMPORTANT]
> **Cybersecurity Perspective:**
> While a `500` error indicates a bug to developers, to a penetration tester it is a **major breakthrough**.
> 
> When injecting special payload characters (such as a single quote `'`) causes the server response to switch from `200 OK` to `500 Internal Server Error`, it proves that:
> 1. User input was **not sanitized** or filtered by the application.
> 2. The input reached the backend interpreter/database engine and broke the query logic syntax.
> 3. The application lacks error handling, signaling a very high probability of **SQL Injection (SQLi)** or command execution flaws.

```sql
-- Normal behavior (Server returns 200 OK):
SELECT * FROM users WHERE username = 'Ali'

-- Injecting a single quote ' causes a syntax error (Server crashes with 500 Internal Error):
SELECT * FROM users WHERE username = '''
```

---

### 6. `502 Bad Gateway` (Gateway / Proxy Failures)
Returned when an edge server (such as an Nginx reverse proxy, load balancer, or Cloudflare) receives an invalid response from the upstream backend server.

> [!NOTE]
> Often observed during **Denial of Service (DoS)** testing or heavy automated scanning when backend database connections become overwhelmed and crash.

---

## 🧪 Practical Scenarios & Analysis

<details>
<summary><b>🔹 Scenario 1: Bypassing 403 Forbidden on Sensitive Files</b></summary>

<br>

* **Condition:** During directory enumeration, an automated scanner discovers `/admin/backup.zip` returning a `403 Forbidden` status code.
* **Analysis:** The `403` code confirms that `backup.zip` exists on the server, but access is blocked by permission rules.
* **Testing Methodology:**
  1. Test path manipulation payloads (e.g., `/admin/./backup.zip`, `/admin/..;/backup.zip`).
  2. Inject internal proxy headers (`X-Forwarded-For: 127.0.0.1`, `X-Custom-IP-Authorization: 127.0.0.1`).
  3. Change the request method (e.g., from `GET` to `POST` or `HEAD`).
</details>

<details>
<summary><b>🔹 Scenario 2: Identifying Injection Vulnerabilities via 500 Status Code</b></summary>

<br>

* **Condition:** Submitting a single quote `'` in a search field changes the server response from `200 OK` to `500 Internal Server Error`.
* **Deduction:** The application passed the single quote `'` directly into the backend database query without sanitization, breaking query syntax and causing an unhandled exception.
* **Conclusion:** High probability of **SQL Injection (SQLi)**. The next step is crafting valid SQL syntax payloads (e.g., `' OR '1'='1`) to confirm full exploitation.
</details>
