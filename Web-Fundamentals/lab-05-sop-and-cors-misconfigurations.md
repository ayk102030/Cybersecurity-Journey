# 🌐 Lab 05: Same-Origin Policy (SOP) & CORS Misconfigurations

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-05-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

The **Same-Origin Policy (SOP)** is a foundational browser security mechanism designed to isolate potentially malicious documents from safe ones. While SOP prevents one origin from reading data residing on another, modern web applications frequently require legitimate cross-domain resource sharing via **Cross-Origin Resource Sharing (CORS)**. This lab explores SOP mechanics, compares CORS against CSRF and `SameSite` cookies, analyzes high-impact CORS misconfigurations, and documents practical hands-on scenarios along with technical evaluations.

---

## 🌐 Understanding the Same-Origin Policy (SOP)

SOP restricts client-side scripts running in one origin from reading responses from a resource on a different origin.

### ⚙️ What Constitutes an "Origin"?
An origin is uniquely defined by a tuple of three components:

$$\text{Origin} = \text{Protocol} + \text{Domain} + \text{Port}$$

If any single component differs between two URLs, browsers classify them as **Cross-Origin**, enforcing strict SOP boundaries.

### 📊 Origin Evaluation Table
*(Base Target URL: `https://example.com:443`)*

| Tested URL | Same Origin? | Reason for Classification |
| :--- | :---: | :--- |
| `https://example.com/about.html` | ✅ **Yes** | Protocol (`https`), Domain (`example.com`), and Port (`443`) match. Path differences do not alter the origin. |
| `http://example.com` | ❌ **No** | Protocol mismatch (`http` vs `https`). |
| `https://api.example.com` | ❌ **No** | Domain mismatch (Subdomain `api.example.com` vs root domain). |
| `https://example.com:8080` | ❌ **No** | Port mismatch (`8080` vs `443`). |

> [!WARNING]
> **SOP Execution vs. Read Distinction:**  
> SOP primarily prevents **reading** cross-origin responses via JavaScript (e.g., `fetch()` or `XMLHttpRequest`). It does **not** always prevent sending requests or embedding external resources (e.g., `<img src="...">`, `<script src="...">`).

---

## ⚔️ Comparative Analysis: CORS vs. CSRF vs. SameSite

| Concept | Core Vulnerability / Need Addressed | Defensive Mechanism | Primary Goal |
| :--- | :--- | :--- | :--- |
| **CORS** | SOP blocks legitimate cross-domain API calls. | Server exposes `Access-Control-Allow-*` HTTP headers. | Explicitly authorize cross-origin **data reads**. |
| **CSRF** | Browsers automatically attach ambient session cookies to cross-origin requests. | Anti-CSRF Tokens, re-authentication, `SameSite` flags. | Prevent execution of **unintended actions**. |
| **`SameSite`** | Cookies default to global cross-origin attachment. | Cookie attributes (`Strict`, `Lax`, `None`). | Restrict automatic cookie transmission to cross-site contexts. |

---

## 🛠️ CORS Misconfigurations & Attack Vectors

A CORS misconfiguration occurs when developers implement overly permissive CORS rules, effectively poking holes in browser SOP enforcement.

### 1. Arbitrary Origin Reflection (with Credentials)
* **The Flaw:** The server dynamically reflects any value sent in the incoming `Origin` header into `Access-Control-Allow-Origin` while also setting `Access-Control-Allow-Credentials: true`.
  ```http
  HTTP/1.1 200 OK
  Access-Control-Allow-Origin: [https://attacker.com](https://attacker.com)
  Access-Control-Allow-Credentials: true
  ```
* **Exploitation:** An attacker hosts a malicious script on `https://attacker.com`. When an authenticated victim visits the page, the script issues an authenticated background request to `https://target.com`. The server approves `attacker.com`, allowing the attacker's script to read the victim's private response payload (PII, API keys, emails).

### 2. Trusted `null` Origin Exploitation
* **The Flaw:** Developers whitelist `null` origins in CORS configuration to accommodate local files (`file://`) or sandboxed `iframe` elements.
  ```http
  HTTP/1.1 200 OK
  Access-Control-Allow-Origin: null
  Access-Control-Allow-Credentials: true
  ```
* **Exploitation:** Attackers trigger a `null` origin request using a sandboxed `iframe` (`<iframe sandbox="allow-scripts allow-top-navigation" src="...">`) to bypass domain-matching checks and extract sensitive application data.

### 3. Wildcard Origin (`Access-Control-Allow-Origin: *`)
* **The Flaw:** Exposing public resources to all origins.
* **Impact:** Browsers explicitly block the combination of `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`. However, if endpoint data relies on IP-based trust or unauthenticated internal network access, an attacker can still read sensitive response bodies from external sites.

---

## 🛡️ Defensive Remediation

* **Avoid Dynamic Reflection:** Never reflect arbitrary incoming `Origin` headers directly into response headers.
* **Maintain Strict Whitelists:** Validate incoming `Origin` values against an explicit, server-side whitelist of trusted domains.
* **Prohibit `null` Trust:** Never allow `Access-Control-Allow-Origin: null`.
* **Scope CORS Exposure:** Restrict CORS headers exclusively to non-sensitive endpoints requiring public cross-domain access.

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Scenario 1: Same-Origin Policy (SOP) Evaluation

* **Scenario Question:**
  > You have a website hosted at `https://dashboard.site.com`. A JavaScript script executing on `http://dashboard.site.com` attempts to read data from it. Will the browser allow this operation based on the Same-Origin Policy (SOP)? Why?

* **Student Answer:**
  > No, the browser will not allow JavaScript execution/read because the other site has a different domain and does not specify origin.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **Correct Conclusion** *(Access Blocked)* with a technical clarification.
  * **Detailed Assessment:** The browser indeed **blocks** the read operation. However, the precise technical reason is a **protocol mismatch** (`https` operating on port `443` vs `http` operating on port `80`), not a domain mismatch (the domain `dashboard.site.com` is identical in both contexts). Since an Origin is defined as `Protocol + Domain + Port`, any difference in protocol causes the browser to treat them as Cross-Origin under SOP.

---

### 🔹 Scenario 2: Exploiting CORS Arbitrary Origin Reflection

* **Scenario Question:**
  > While inspecting `https://target.com` using Burp Suite, you manually modify the request header to `Origin: https://hacker.com`. The server responds with:
  > ```http
  > HTTP/1.1 200 OK
  > Access-Control-Allow-Origin: [https://hacker.com](https://hacker.com)
  > Access-Control-Allow-Credentials: true
  > ```
  > What vulnerability is present here? How would you exploit it as a penetration tester to exfiltrate user data?

* **Student Answer:**
  > Here, `hacker.com` has become trusted by the server. I can execute JavaScript code to steal session data by sending requests to the server for this purpose.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Exemplary & Accurate Response.**
  * **Vulnerability Classification:** **CORS Arbitrary Origin Reflection with Credential Support**.
  * **Exploitation Workflow:**
    1. The attacker hosts a malicious JavaScript payload on `https://hacker.com`.
    2. The script issues a background `fetch()` request to `https://target.com/api/user/profile` with `credentials: 'include'`.
    3. When an authenticated victim visits `https://hacker.com`, their browser automatically attaches their session cookies to `target.com`.
    4. Because `target.com` responds with `Access-Control-Allow-Origin: https://hacker.com` and `Access-Control-Allow-Credentials: true`, the browser permits `hacker.com` JavaScript to read the response and exfiltrate the victim's private data to an attacker-controlled server.
