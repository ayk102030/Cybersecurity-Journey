# 🌐 Lab 08: Session Management, Incomplete Logout & Session Fixation

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-08-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

Because HTTP is a **stateless** protocol, servers do not natively remember client identity between requests. **Session Management** bridges this gap by maintaining server-side state associated with a unique client identifier (Session ID). This lab examines the end-to-end session lifecycle, key audit criteria for penetration testers, and two high-risk session vulnerabilities: **Incomplete Session Termination** (Client-Side Logout Only) and **Session Fixation**.

---

## 📊 Quick Reference Table

| Concept / Vulnerability | Mechanism | Root Cause | Primary Mitigation |
| :--- | :--- | :--- | :--- |
| **Session Lifecycle** | Issue, store, track, and destroy Session IDs. | N/A (Standard Auth Model) | Secure cookie flags (`HttpOnly`, `Secure`, `SameSite`). |
| **Incomplete Logout** | Session remains valid on server after logout. | Client-side cookie deletion without backend invalidation. | Destroy session entry in server-side Session Store. |
| **Session Fixation** | Server reuses pre-auth Session ID post-login. | Failure to regenerate Session ID upon privilege escalation. | Call `session_regenerate_id(true)` upon successful login. |

---

## ⚙️ Session Lifecycle (Step-by-Step)

```
[ Client / Browser ]                                  [ Server / Backend ]
         |                                                     |
         | --- 1. POST /login (username & password) ---------> | (Authenticates credentials)
         |                                                     | (Generates unique Session ID)
         |                                                     | (Maps Session ID -> User in RAM/Redis)
         | <-- 2. 200 OK (Set-Cookie: PHPSESSID=f83a...) ----- |
         |                                                     |
  (Stores Cookie)                                              |
         |                                                     |
         | --- 3. GET /profile (Cookie: PHPSESSID=f83a...) --> | (Looks up PHPSESSID in Session Store)
         | <-- 4. 200 OK (Returns profile data) -------------- | (Returns user data if matched)
```

### 1. Authentication Phase
The user submits valid credentials via a standard HTTP request:

```http
POST /login HTTP/1.1
Host: target.com

username=admin&password=Password123
```

### 2. Generation & Mapping Phase
* The server validates credentials against the user database.
* Upon successful authentication, the server creates a unique session record inside its memory (e.g., RAM or Redis Session Store).
* The server generates a cryptographically secure, random string called a **Session ID**.
* The server binds/maps this Session ID to the authenticated user's profile data.

### 3. Delivery Phase
The server returns the HTTP response to the client browser, attaching the identifier via the `Set-Cookie` header:

```http
HTTP/1.1 200 OK
Set-Cookie: PHPSESSID=f83a91b2c4e5...; Path=/; Secure; HttpOnly
```

### 4. Tracking Phase
The browser automatically stores the cookie. For subsequent requests (e.g., accessing `/profile`), the browser automatically sends the identifier back in the header:

```http
GET /profile HTTP/1.1
Host: target.com
Cookie: PHPSESSID=f83a91b2c4e5...
```
The server compares the incoming `PHPSESSID` value against its active Session Store. Upon a match, it renders the authorized user data.

---

## 🛡️ Penetration Tester Assessment Criteria

When auditing session management mechanisms, penetration testers evaluate three core dimensions:

> [!NOTE]
> 1. **Storage Location:** Is session state maintained strictly in a secure server-side Session Store, or is user state serialized and trusted on the client side?
> 2. **Randomness & Predictability:** Is the Session ID complex and cryptographically secure, or is it predictable (e.g., based on sequential numbers, timestamps, or predictable user attributes)?
> 3. **Session Termination (Invalidation):** Does logging out explicitly invalidate/destroy the session record in the server's Session Store, or does it merely instruct the browser to delete the cookie locally?

---

## 🛠️ Vulnerability Deep Dive

### 1. Incomplete Session Termination (Client-Side Logout)

> [!CAUTION]
> **Root Cause:** The logout handler clears the browser cookie (e.g., issuing an expired `Set-Cookie` header) but fails to destroy or invalidate the active session record inside the backend **Session Store**.
> 
> **Technical Reality:** Active sessions are tracked in memory structures like Session Stores, not evaluated from text log files (which are archival audit records).
> 
> **Impact:** If an attacker steals a victim's Session ID (via XSS, network eavesdropping, or log leaks), they can inject the token into their browser and maintain full account access indefinitely, regardless of whether the victim clicked "Logout".

---

### 2. Session Fixation Vulnerability

> [!WARNING]
> **Mechanism:** In a secure setup, an unauthenticated user may receive a temporary Session ID. However, immediately upon successful login, the server **must** destroy the pre-authentication identifier and issue a brand-new Session ID.
> 
> **Session Fixation** occurs when the developer fails to issue a new Session ID post-login, allowing an attacker to pre-define and "fixate" a Session ID onto a victim prior to authentication.

#### 💣 Step-by-Step Exploitation Workflow
1. **Preparation:** The attacker visits `https://target.com` while unauthenticated. The server assigns them a temporary pre-login Session ID: `ATTACKER_SESSION_123`.
2. **Fixation:** The attacker induces the victim to visit a login link containing this pre-set Session ID (via link parameter, CRLF injection, or XSS):  
   `https://target.com/login?PHPSESSID=ATTACKER_SESSION_123`
3. **Victim Login:** The victim opens the link and enters valid credentials to log in.
4. **Elevation:** The server authenticates the victim but retains the old identifier (`ATTACKER_SESSION_123`), associating the victim's account with the attacker's known token.
5. **Exploitation:** The attacker sets `PHPSESSID=ATTACKER_SESSION_123` in their own browser and immediately gains full access to the victim's account.

#### 🛡️ Defensive Remediation
Enforce session identifier regeneration in application code immediately following successful authentication.

* **PHP Implementation:**
  ```php
  session_regenerate_id(true); // Generates a new Session ID and deletes the old session record from the server
  ```

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Scenario 1: Auditing Logout Invalidation Mechanics

* **Question:**
  > During a web application assessment, you log in and capture your assigned Session ID. You copy the cookie value, then click the "Logout" button inside the web app. Afterwards, using Burp Suite Repeater, you manually re-send a request to `GET /profile` attaching the previously saved Session ID cookie.  
  > If the server responds with `200 OK` and renders full account data, what does a penetration tester deduce from this behavior, and what risk does it present?

* **Student Answer:**
  > As soon as an attacker gets hold of the Session ID (via XSS or network sniffing), they can set it in their browser and access the account and pages completely without needing the username or password.  
  > **Defensive Flaw:** The server committed a client-side-only logout error by deleting the cookie from the browser, while continuing to treat the session as valid in its backend storage. The server does not read active sessions from text log files (logs are archival records); the true storage location is the Session Store.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Accurate Analysis.**
  * **Vulnerability Name:** **Incomplete Session Invalidation / Broken Session Termination**.
  * **Security Risk:** High. Reusable session tokens allow session hijacking to persist long after the legitimate user has signed out.

---

### 🔹 Scenario 2: Session Fixation Proof of Concept (PoC)

* **Question:**
  > You are testing a banking application. Before entering credentials on the login page, you observe your assigned pre-authentication Session ID is `XYZ123`. After successfully logging into your account, you inspect your HTTP requests in Burp Suite and observe that your post-authentication Session ID is still `XYZ123`.  
  > 1. What vulnerability exists here?  
  > 2. How would you construct a full Proof of Concept (PoC) scenario to demonstrate its risk against another user?

* **Student Answer:**
  > 1. **Vulnerability:** The server does not generate a new session after login, which can lead to manipulation by a hacker (Session Fixation).  
  > 2. **PoC Scenario:** The hacker sends the pre-login session ID to the victim and tricks them into logging in using that exact session. Once logged in, the account and data become accessible to the hacker using the known session ID.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Accurate & Well-Structured PoC.**
  * **Vulnerability Classification:** **Session Fixation**.
  * **PoC Execution Sequence:**
    1. Attacker obtains an unauthenticated session ID (`XYZ123`).
    2. Attacker forces `XYZ123` onto the victim's session context via a crafted link.
    3. Victim completes authentication on `bank.com`.
    4. Because `XYZ123` remains unchanged post-login, it becomes elevated to an authenticated session.
    5. Attacker accesses the victim's bank account using `XYZ123`.

---
