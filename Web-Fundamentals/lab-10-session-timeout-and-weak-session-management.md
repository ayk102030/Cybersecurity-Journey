# 🌐 Lab 10: Session Timeout & Weak Session Management

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-10-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

Session timeout mechanisms and control policies serve as the final line of defense against session hijacking and unauthorized account persistence. Proper session lifecycle management dictates how long a session token remains valid, how concurrent connections are constrained, and how active sessions are invalidated upon critical account events. This lab explores session timeout types, common session management flaws, and practical security assessments.

---

## 📊 Quick Reference Table

| Mechanism / Flaw | Core Definition | Primary Security Risk |
| :--- | :--- | :--- |
| **Idle Timeout** | Max time a session stays active without new client requests. | Unattended public devices remain vulnerable to physical takeover. |
| **Absolute Timeout** | Fixed lifespan of a session since creation, regardless of activity. | Stolen sessions remain valid indefinitely if kept active via periodic requests. |
| **Concurrent Sessions** | Unrestricted simultaneous active logins across multiple clients. | Allows adversaries to maintain silent secondary access alongside legitimate users. |
| **Session Revocation** | Invalidation of all active tokens upon sensitive account state changes. | Adversaries retain persistent access even after a victim changes their password. |
| **Predictable Session IDs** | Generating tokens using deterministic patterns or weak algorithms. | Enables session token enumeration and brute-force prediction attacks. |

---

## ⏳ Types of Session Timeouts

A secure authentication system must enforce two complementary session expiration mechanisms:

### 1. Idle Timeout
* **Concept:** Specifies the maximum allowable time a session can remain active without the client issuing any new HTTP requests (e.g., 15 minutes of inactivity).
* **Security Risk:** If omitted, an unattended workstation left open in a public space allows nearby unauthorized individuals to immediately use the account.

### 2. Absolute Timeout
* **Concept:** Establishes a fixed hard limit on the total lifetime of a session measured from the exact moment of creation, regardless of user activity (e.g., 8 hours total lifespan).
* **Security Risk:** If omitted, an attacker who steals a session cookie can maintain continuous access indefinitely simply by sending periodic automated requests to prevent the idle timeout from triggering.

---

## ⚙️ Indicators of Weak Session Management

Penetration testers analyze server-side logic to uncover critical weaknesses in session handling:

1. **Unlimited Concurrent Sessions:**  
   Allowing a single user account to authenticate simultaneously across dozens of different devices, IP addresses, or browsers without applying limits, session management dashboards, or security notifications.

2. **Session Revocation Failure:**  
   Failing to immediately invalidate and revoke all active session tokens on the server side when critical security events occur—such as changing passwords, resetting credentials, or enabling Two-Factor Authentication (2FA).

3. **Predictable Session IDs:**  
   Generating session identifiers using weak, deterministic algorithms based on known or guessable inputs. Examples include:
   * `Base64(Username + Timestamp)`
   * Sequential or incremental values (e.g., `SESS_001`, `SESS_002`)

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Field Scenario: Evaluating Session Revocation & Concurrency Handling

> [!CAUTION]
> **Field Condition:**  
> During a security assessment, you log into a user account simultaneously using two distinct browsers (**Firefox** and **Chrome**).  
> 1. From inside the **Firefox** session, you navigate to Account Settings and perform a **Password Change**.  
> 2. Immediately afterward, you switch to **Chrome** (which still uses the old session cookie issued prior to the password change) and click a link to `/profile`.  
> 3. The server processes the Chrome request, returns an `HTTP 200 OK` response, and allows the session in Chrome to function normally.

* **Questions:**
  1. What programmatic flaw exists in the session management logic here?
  2. How does this vulnerability affect a victim whose account was compromised and who changed their password to regain control?

---

* **Student Analysis:**
  > 1. **Programmatic Flaws:**  
  >    * **Session Revocation Failure:** A critical error where old sessions do not expire after a password change.  
  >    * **Unlimited Concurrent Sessions:** In addition, allowing simultaneous active logins across multiple browsers from the start without restrictions.  
  >    * **Required Server Behavior:** The server must reject the Chrome request, invalidate the token, and redirect the client to re-authenticate with a new session.  
  >  
  > 2. **Security Impact:**  
  >    * The attacker retains persistent access (**Persistence**) inside the stolen account via the compromised session token.  
  >    * Even if the victim discovers the breach and immediately changes their password, the attacker remains logged in because the server fails to invalidate the existing session cookie, rendering the password change ineffective.  
  >    * The victim would have to manually navigate to security settings (if available) to clear active sessions across all devices.

---

* **Technical Evaluation & Breakdown:**
  * **Verdict:** ✅ **100% Accurate & Spot-On Analysis.**
  * **Primary Flaw Identified:** **Session Revocation Failure**. The server must maintain a mechanism that revokes all existing active session tokens immediately upon a password change or reset.
  * **Secondary Flaw Identified:** **Unlimited Concurrent Sessions**. The absence of concurrency limits exacerbates the window of exposure.
  * **Security Impact:** Grants the adversary continuous persistence within the victim's account, neutralizing the primary remediation mechanism (password change) that users rely on during an account compromise.
