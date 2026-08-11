# 🌐 Lab 17: Encoding Types Cheatsheet — Complete Field & Reference Guide for Web Pentesting

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-17-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

**Encoding** is a fundamental mechanism in web technologies designed to ensure data integrity during transmission across different protocols, contexts, and application layers. 

For Web Application Penetration Testers and Security Analysts, mastering encoding techniques is crucial for:
1. **Context Breaking:** Escaping constrained input fields (HTML, JavaScript, SQL, JSON).
2. **Filter & WAF Evasion:** Bypassing signature-based security inspection layers.
3. **Data Manipulation:** Tampering with encoded session states, tokens, and parameter values.

---

## 🛠️ Detailed Encoding Types & Field Application

### 1. URL Encoding (Percent-Encoding)
* **Concept:** Converts special characters into a `%` symbol followed by their 2-digit Hexadecimal ASCII representation to prevent breaking HTTP Request URI structures.
* **Example:**
  * **Original:** `<script>`
  * **Encoded:** `%3Cscript%3E`
* **Target Location:** URL Paths, GET Query Parameters (e.g., `?search=payload`), and `application/x-www-form-urlencoded` POST bodies.
* **Pentesting Scenario:** Bypassing space or character restrictions (`<`, `>`, `'`, `"`, `/`) in GET requests to deliver XSS or SQLi payloads without breaking HTTP request syntax.

---

### 2. HTML Encoding (Entities)
* **Concept:** Replaces reserved HTML characters with pre-defined entity names or numeric values (starting with `&` and ending with `;`) to prevent browser parsing as executable markup.
* **Example:**
  * **Original:** `'`
  * **Named Entity:** `&apos;`
  * **Numeric Entities:** `&#x27;` (Hex) / `&#39;` (Decimal)
* **Target Location:** HTML Document Body and HTML Attribute Values.
* **Pentesting Scenario:** Evaluating defense effectiveness against XSS. If user input is HTML-encoded inside an HTML context, it is secure; however, if reflects inside a JavaScript block or unquoted attribute context without proper escaping, context breaking remains possible.

---

### 3. Base64 Encoding
* **Concept:** Converts binary or text data into an ASCII string format using a 64-character printable alphabet (`A-Z`, `a-z`, `0-9`, `+`, `/`, with `=` used for padding).
* **Example:**
  * **Original:** `{"role": "user"}`
  * **Encoded:** `ZXlKcmIyeGxJam9pInVzZXIifQ==`
* **Target Location:** Session Cookies, HTTP Authorization Headers (e.g., `Basic Auth`), and Authorization Tokens (e.g., JWT payloads).
* **Pentesting Scenario:** Decoding client-side state parameters to identify Privileges, User IDs, or Object References. Modify values (e.g., parameter tampering / IDOR / privilege escalation to `admin`), re-encode, and transmit.

---

### 4. Unicode Encoding (`\uXXXX`)
* **Concept:** Represents characters using a 4-digit hexadecimal code point preceded by the `\u` prefix, matching the global Unicode standard.
* **Example:**
  * **Original:** `'`
  * **Encoded:** `\u0027`
* **Target Location:** REST API endpoints, JSON request payloads, and JavaScript Execution Engines.
* **Pentesting Scenario:** Exploiting **Parser Differentials**. WAFs inspecting raw input streams pass `\u0027` because no literal single quote (`'`) is detected. The backend JSON parser (`JSON.parse`, `json_decode`) automatically decodes `\u0027` to `'`, executing SQLi or XSS.

---

### 5. Double Encoding (Double URL Encoding)
* **Concept:** Applies URL encoding twice sequentially. The `%` symbol generated from the first pass (`%XX`) is encoded in the second pass to `%25`, yielding `%25XX`.
* **Example:**
  * **Original:** `/`
  * **1st Pass:** `%2F`
  * **2nd Pass:** `%252F`
* **Target Location:** URL Query Parameters handling multi-tiered reverse proxies and web application servers.
* **Pentesting Scenario:** Exploiting **Decoding Asymmetry**. The WAF decodes input once (viewing `%2F` as safe text), while the backend decodes it a second time (yielding `/`), bypassing Path Traversal filters (`..%252f..%252f`).

---

### 6. Hex Encoding (`0x...`)
* **Concept:** Translates text/binary data into raw hexadecimal values prefixed with `0x`, allowing database engines to parse the input natively as string constants.
* **Example:**
  * **Original:** `admin`
  * **Encoded:** `0x61646d696e`
* **Target Location:** SQL Injection Payloads within database queries (MySQL / MariaDB).
* **Pentesting Scenario:** Bypassing strict quote filters (`'` or `"`). Converting schema names or string literals into `0x...` allows direct execution within SQL queries without relying on string delimiter quotes.

---

## 📊 Quick-Reference Comparison Matrix

| Encoding Type | Syntax / Prefix | Target Context / Layer | Primary Pentesting Objective |
| :--- | :---: | :--- | :--- |
| **URL Encoding** | `%XX` | HTTP Request URIs & Query Parameters | Prevent URI corruption / Pass special chars in GET |
| **HTML Encoding** | `&name;` / `&#XX;` | HTML Body & Attributes | Analyze XSS defense / Break out of script contexts |
| **Base64** | `A-Z, a-z, 0-9, +, /, =` | Cookies, Headers, JWTs | Decode, tamper session data/privileges, re-encode |
| **Unicode** | `\uXXXX` | JSON Payloads & APIs | Bypass WAF signatures via backend parser decoding |
| **Double Encoding** | `%25XX` | Multi-tier Web Architectures | Exploit WAF/Backend decoding asymmetry (Path Traversal) |
| **Hex Encoding** | `0x...` | Database SQL Queries | Execute SQLi queries without single/double quotes |

