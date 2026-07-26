# 🌐 Lab 03: HTTP Headers & Cybersecurity Implications

## 📌 Overview

In the **HTTP** protocol, **Headers** act as the metadata envelope of an HTTP request or response. While the request body contains the actual data payload, headers define essential parameters such as target destinations, client identities, session credentials, and origin details. For penetration testers, understanding and manipulating headers is crucial for bypassing Web Application Firewalls (WAFs), executing session hijacking, spoofing internal network identities, and discovering logic vulnerabilities.

---

## 📊 Quick Reference Table

| Header | Primary Purpose | Key Cybersecurity Risk / Pentesting Use |
| :--- | :--- | :--- |
| **`Host`** | Specifies domain/server name | Host Header Injection & Reset Poisoning |
| **`User-Agent`** | Identifies client browser & OS | WAF Evasion & Scanner Stealth |
| **`Cookie`** | Stores session identifiers | Session Hijacking & Session Fixation |
| **`Authorization`** | Carries credentials / Bearer Tokens | Broken Authentication & Privilege Escalation |
| **`Referer`** | Tracks traffic source URL | Sensitive Data Leakage in URLs |
| **`X-Forwarded-For`** | Tracks real client IP through proxies | IP Spoofing & Access Control Bypass |

---

## 🛠️ Detailed Technical Breakdown

### 1. `Host` (Target Server Domain)
Specifies the domain name of the server that the client wants to connect to (e.g., `Host: target.com`). This is required in HTTP/1.1 to distinguish between multiple virtual hosts hosted on a single IP address.

> [!WARNING]
> **Host Header Injection:**
> If a web application relies on the `Host` header to dynamically generate password reset links, an attacker can modify this header:
> ```http
> POST /password-reset HTTP/1.1
> Host: attacker.com
> ```
> The generated reset link emailed to the victim will point to `http://attacker.com/reset?token=XYZ`, causing the victim's sensitive token to be sent directly to the attacker's server.

---

### 2. `User-Agent` (Client & Browser Identity)
Informs the server about the client application, operating system, vendor, and software version making the request.

> [!TIP]
> **WAF Evasion & Stealth:**
> Automated scanners (such as `Nmap`, `Burp Suite`, `SQLmap`, or `Gobuster`) broadcast default `User-Agent` strings that WAFs actively detect and block. Penetration testers routinely customize the `User-Agent` header to mimic legitimate browser strings (e.g., Chrome on iOS/Android) to evade signature-based filtering.

---

### 3. `Cookie` (Session Identifiers)
Because HTTP is a **stateless** protocol, servers use the `Cookie` header to store session identifiers (such as `PHPSESSID` or `JSESSIONID`). The client includes this header in every subsequent request to maintain an authenticated session state.

> [!CAUTION]
> **Session Hijacking:**
> If an attacker steals a victim's active session cookie (e.g., via Cross-Site Scripting - XSS or network sniffing), they can inject the cookie into their own browser to impersonate the victim completely without needing account credentials.

---

### 4. `Authorization` (Authentication Credentials & Tokens)
Transmits credentials or access tokens (such as HTTP Basic Auth, OAuth, or Bearer JWT tokens) to authenticate a user on modern web applications and APIs.

```http
GET /api/v1/user/profile HTTP/1.1
Host: target.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> [!IMPORTANT]
> **Broken Access Control:**
> Applications must strictly validate token signatures and user privileges on every request. Weak validation allows attackers to manipulate JWT payloads (e.g., altering `user_id` or setting `"alg": "none"`) to achieve unauthorized privilege escalation.

---

### 5. `Referer` (Traffic Source URL)
Identifies the address of the previous web page from which the current request originated (e.g., indicating the user clicked a link on Google to reach the site).

> [!NOTE]
> **Sensitive Data Leakage:**
> If a application passes sensitive information inside URL parameters (e.g., `https://target.com/reset?token=12345`), clicking an external link on that page will send the full URL including the sensitive token to the external site inside the `Referer` header.

---

### 6. `X-Forwarded-For` (Real Client IP)
When traffic passes through intermediary servers like proxies, load balancers, or reverse proxies (e.g., Nginx, Cloudflare), the server sees the proxy's IP address instead of the client's. The `X-Forwarded-For` header is appended by proxies to preserve the original client IP address.

```http
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-For: 127.0.0.1
```

> [!CAUTION]
> **IP Spoofing & Access Control Bypass:**
> Applications that rely blindly on `X-Forwarded-For` to grant access to internal management interfaces (e.g., restricting `/admin` access to localhost `127.0.0.1`) can be tricked into granting unauthorized access by injecting `X-Forwarded-For: 127.0.0.1`.

---

## 🧪 Practical Scenarios & Analysis

<details>
<summary><b>🔹 Scenario 1: Bypassing WAF Blocklists via User-Agent Modification</b></summary>

<br>

* **Condition:** An automated directory fuzzing scan gets blocked instantly by a target Web Application Firewall (WAF).
* **Analysis:** The WAF's rule engine detected signatures belonging to default automated tools in the `User-Agent` header (e.g., `User-Agent: Gobuster/3.1.0`).
* **Remediation & Testing Technique:** Modify the tool's settings to send a standard mobile browser string, such as:  
  `User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 16_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.5 Mobile/15E148 Safari/604.1`
</details>

<details>
<summary><b>🔹 Scenario 2: Bypassing Restricted Administrative Panels via IP Spoofing</b></summary>

<br>

* **Condition:** Accessing `/admin/console` returns `403 Forbidden` with a message stating: *"Access allowed only from internal network IPs."*
* **Analysis:** The application attempts to enforce access control based on IP validation using reverse proxy headers.
* **Remediation & Testing Technique:** Inject trusted internal IP headers to spoof localhost identity:  
  * `X-Forwarded-For: 127.0.0.1`
  * `X-Originating-IP: 127.0.0.1`
  * `X-Remote-IP: 127.0.0.1`
</details>
