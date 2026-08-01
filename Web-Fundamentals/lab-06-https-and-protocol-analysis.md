# 🌐 Lab 06: HTTPS & Protocol Analysis (HTTP/1.1 vs HTTP/2)

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-06-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

This lab examines the **HTTPS** protocol and transport-layer security from a web penetration testing perspective. It covers TLS/SSL interception techniques, common misconfigurations, reconnaissance via SNI, Mixed Content vulnerabilities, and HSTS enforcement. Additionally, it provides a deep-dive architecture comparison between **HTTP/1.1** (text-based) and **HTTP/2** (binary-based), highlighting core attack vectors such as **HTTP Request Smuggling** and **HTTP/2 Downgrading (H2.CL / H2.TE)**.

---

## 📊 Quick Reference Table

| Feature / Concept | HTTP/1.1 | HTTP/2 | HTTPS Layer Impact |
| :--- | :--- | :--- | :--- |
| **Data Format** | Plain Text (Human-readable) | Binary Frames | Encrypted Payload via TLS |
| **Delimiters** | CRLF (`\r\n`), Headers | Binary Frame Headers + Stream IDs | N/A (Operates below HTTP) |
| **Multiplexing** | ❌ No (Sequential / Pipelining) | ✅ Yes (Concurrent Streams) | N/A |
| **Primary Risk** | Classic Request Smuggling | Downgrade Attacks (H2.CL/H2.TE) | Weak Ciphers, Mixed Content, SSL Stripping |
| **Session Control** | `Content-Length` / `Transfer-Encoding` | Explicit Frame Header Length | Certificate Authentication |

---

## 🛡️ HTTPS Mechanics & Penetration Testing Dimensions

HTTPS encapsulates standard HTTP traffic inside an encrypted **TLS (Transport Layer Security)** or **SSL** tunnel, providing **Encryption** (Data in Transit protection) and **Authentication** (via Certificate Authorities).

### 1. Traffic Interception & Burp Suite
When routing HTTPS traffic through an intercepting proxy (such as Burp Suite), the browser detects an untrusted Man-in-the-Middle (MiTM) and displays a security error (e.g., `NET::ERR_CERT_AUTHORITY_INVALID`).

> [!TIP]
> **Remediation for Testers:** To intercept and decrypt HTTPS traffic seamlessly, the penetration tester must export Burp Suite's Custom Root CA Certificate (`CA Certificate`) and install/trust it within the browser or OS certificate store.

### 2. TLS/SSL Misconfigurations
Penetration testers evaluate the underlying TLS implementation for structural weaknesses:
* **Outdated Protocols:** Support for SSLv3, TLS 1.0, or TLS 1.1 exposes the server to cryptographic attacks (e.g., POODLE, BEAST).
* **Weak Cipher Suites:** Short key lengths or obsolete ciphers allow attackers to break encryption or decrypt captured traffic.
* **Certificate Inspection (SAN):** Certificates containing hidden internal subdomains in the **Subject Alternative Name (SAN)** field reveal unlisted attack surfaces during reconnaissance.

### 3. Server Name Indication (SNI) Reconnaissance
During the initial TLS handshake, the client sends an unencrypted `Client Hello` packet containing the **SNI header**, which exposes the destination domain in plain text before encryption is established.

> [!NOTE]
> **Pentesters' Advantage:** Passive network sniffing of SNI headers allows testers to identify backend endpoints and subdomains even when traffic payloads are fully encrypted.

### 4. Mixed Content Vulnerabilities
Occurs when an HTTPS-secured page loads external assets (e.g., JavaScript, CSS) over an unencrypted `http://` connection.

> [!CAUTION]
> **Active Mixed Content Risk:** If an HTTPS page imports an unencrypted script (`http://domain.com/script.js`), a network adversary (MiTM) can intercept the HTTP request, inject arbitrary JavaScript code, and execute it within the trusted HTTPS context of the victim's session.

### 5. HTTP Strict Transport Security (HSTS)
HSTS forces browsers to communicate exclusively over HTTPS via the response header:
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

> [!IMPORTANT]
> **Absence Impact:** If HSTS is omitted, an attacker on the same network can perform **SSL Stripping**, downgrading the victim's connection from `https://` to plain `http://` and capturing sensitive credentials.

---

## ⚙️ Protocol Architecture: HTTP/1.1 vs HTTP/2

```
+-------------------------------------------------------------------+
|                        HTTP/1.1 Protocol                          |
|  [Text Request 1] \r\n\r\n [Text Request 2] \r\n\r\n (CRLF Based)  |
+-------------------------------------------------------------------+
                                 vs
+-------------------------------------------------------------------+
|                         HTTP/2 Protocol                           |
|  [Stream 1: Binary Frame] [Stream 3: Binary Frame] [Stream 1...]  |
+-------------------------------------------------------------------+
```

### 1. HTTP/1.1 (Text-Based Architecture)
* **Text Delimiters:** Uses explicit CRLF characters (`\r\n`) to demarcate lines, with headers terminating at double CRLF (`\r\n\r\n`).
* **Pipelining & Keep-Alive:** Multiple requests share a single TCP connection sequentially.
* **Vulnerability Root Cause:** Because parsing relies on header string interpretation (`Content-Length` vs `Transfer-Encoding`), discrepancies between front-end reverse proxies and backend servers lead to **HTTP Request Smuggling**.

### 2. HTTP/2 (Binary Frame Architecture)
* **Binary Framing:** Requests and responses are split into binary-encoded frames with exact byte lengths specified inside the **Frame Header**.
* **Multiplexing & Stream IDs:** Requests interleave concurrently over a single TCP connection, tagged with unique **Stream IDs**.
* **Native Defense:** Pure HTTP/2 environments are immune to classic Request Smuggling because parsing relies on hardcoded frame sizes rather than ambiguous text delimiters.

### 3. HTTP/2 Downgrading Attacks (H2.CL / H2.TE)
Most modern architectures deploy an HTTP/2 front-end proxy paired with an older HTTP/1.1 backend server.

> [!WARNING]
> **Exploitation Vector:** An attacker sends a malicious HTTP/2 request containing embedded HTTP/1.1 smuggling payloads. When the front-end proxy **downgrades** the request to HTTP/1.1 for backend forwarding, the binary frames are translated into text headers. This transformation exposes parsing discrepancies in the backend, triggering **H2.CL** or **H2.TE** Request Smuggling.

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Part A: HTTPS Security Scenarios

#### Scenario 1: Interception Warning Bypass
* **Question:**
  > You configured your browser to route traffic through Burp Suite to test an HTTPS application. However, the browser completely refuses the connection and displays a security warning preventing access. What practical step must you execute in your browser to bypass this restriction and inspect HTTPS traffic?

* **Student Answer:**
  > Change the authentication certificate...

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **Accurate Core Concept** with technical elaboration.
  * **Refinement:** The required action is to **export Burp Suite's Root CA Certificate (`PortSwigger CA`) and import it into the browser's Trusted Root Certification Authorities store**. This authorizes Burp Suite as a trusted issuer, allowing it to generate on-the-fly TLS certificates without browser warnings.

#### Scenario 2: Active Mixed Content Exploitation
* **Question:**
  > While auditing an HTTPS-encrypted e-commerce site enforced with HSTS, you notice that a form submission button fetches an external JavaScript file via `http://`. What vulnerability is present? How can an attacker on the same local network exploit this despite HTTPS encryption?

* **Student Answer:**
  > Intercept the request and modify the JavaScript file to add a malicious file.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **Exemplary & Spot-on Analysis.**
  * **Vulnerability Classification:** **Active Mixed Content**.
  * **Exploitation Workflow:** An adversary performing a Man-in-the-Middle (MiTM) attack intercepts the unencrypted `http://` request for the script file, injects malicious JavaScript (e.g., a credential/credit card skimmer), and returns it to the browser. Because the host page trusts the script context, the injected payload executes within the secure `https://` session, completely bypassing transport encryption guarantees.

---

### 🔹 Part B: Protocol Analysis Scenarios

#### Question 1: Request Smuggling Immunity Factors
* **Question:**
  > What fundamental technical reason makes HTTP/1.1 vulnerable to request boundaries overlap (Request Smuggling), whereas pure HTTP/2 is inherently immune to this classic attack vector?

* **Student Answer:**
  > Because the first relies on text to define the start and end of a request, while the second is strictly defined inside the frame header and not just regular text, so there is no ambiguity between servers in the second.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Technically Accurate.**
  * **Detailed Explanation:** HTTP/1.1 relies on human-readable text delimiters (`CRLF`) and competing header fields (`Content-Length` vs. `Transfer-Encoding`), creating parsing ambiguities between proxies and backends. Pure HTTP/2 uses binary-encoded framing where each frame contains an explicit, immutable length field in its header alongside unique **Stream IDs**, eliminating delimiter-based manipulation entirely.

#### Question 2: HTTP/2 Downgrading Attack Vectors
* **Question:**
  > Imagine testing an application where the front-end accepts HTTP/2, but the backend server processes requests using HTTP/1.1. How can an attacker exploit this architecture to execute an HTTP Request Smuggling attack, and what is the technical name of this technique?

* **Student Answer:**
  > Send a normal request to HTTP/2, but exploit the arrival of the request to HTTP/1 when it is downgraded, injecting the information here. Its name is H2.CL.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Correct & Professional Answer.**
  * **Technique Name:** **HTTP/2 Downgrading Attack (H2.CL / H2.TE Smuggling)**.
  * **Exploitation Mechanics:** The attacker crafts an HTTP/2 request containing smuggled HTTP/1.1 headers or bodies. When the front-end reverse proxy translates (downgrades) the binary HTTP/2 frame into a text-based HTTP/1.1 stream for the backend, the injected headers are rewritten into plain text, causing the backend server to misinterpret request boundaries and process a smuggled secondary request.
