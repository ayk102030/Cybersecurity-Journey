# 🌐 Lab 07: Data Length Specification: Content-Length vs Transfer-Encoding

![Category](https://img.shields.io/badge/Category-Web_Security-blue?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-07-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

## 📌 Overview

In the **HTTP/1.1** protocol, when a client sends a request containing a body (such as a `POST` request), the receiving server must determine the exact boundary of the payload to know where the current request ends and where the subsequent request begins. HTTP/1.1 defines two primary headers for specifying payload length: **`Content-Length`** and **`Transfer-Encoding: chunked`**. Discrepancies in how front-end proxies and backend servers parse these headers lead to HTTP Request Desynchronization and **HTTP Request Smuggling** vulnerabilities.

---

## 📊 Quick Reference Table

| Specification Header | Processing Mechanism | Termination Indicator | Primary Vulnerability Context |
| :--- | :--- | :--- | :--- |
| **`Content-Length` (CL)** | Specifies total payload size in explicit decimal bytes. | Exact byte count reached following `\r\n\r\n`. | **CL.TE Smuggling** (Front-end uses CL, Back-end uses TE). |
| **`Transfer-Encoding` (TE)** | Streams payload in dynamic, hex-sized data blocks (*chunks*). | Explicit zero-sized chunk (`0\r\n\r\n`). | **TE.CL Smuggling** (Front-end uses TE, Back-end uses CL). |

---

## 🛠️ Detailed Technical Breakdown

### 1. `Content-Length` (CL)
Specifies the exact size of the request body in bytes using a decimal integer.

```http
POST /search HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=pentest
```

> [!NOTE]
> **Server Execution Flow:**  
> The server reads headers until it encounters the double CRLF sequence (`\r\n\r\n`). Reading `Content-Length: 11`, it consumes exactly 11 bytes (`q=pentest`) from the socket buffer and marks the request as complete.

---

### 2. `Transfer-Encoding: chunked` (TE)
Used when the total payload size is dynamically generated or unknown prior to transmission (e.g., streaming data, large file uploads).

#### Chunked Format Anatomy
* Each chunk starts with its size written in **Hexadecimal**, followed by `\r\n`.
* The chunk data payload follows, terminated by `\r\n`.
* The final terminating block is a zero-length chunk: `0\r\n\r\n`.

```http
POST /search HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

8
q=pentes
3
t12
0
```

---

## ⚙️ RFC 7230 & The Root Cause of Desynchronization

Standard HTTP/1.1 specifications (**RFC 7230**) explicitly state:

> *"If a message is received with both a `Transfer-Encoding` header field and a `Content-Length` header field, the `Content-Length` MUST be ignored by the receiver."*

### ⚔️ Desynchronization Scenarios (HTTP Request Smuggling)
When multi-tier web architectures (Front-end Reverse Proxy / CDN + Backend Server) handle dual-header requests inconsistently, request desynchronization occurs.

```
                  +-------------------+                 +-------------------+
                  |                   |  Processes CL   |                   |
                  |     Front-End     |---------------> |      Backend      |
                  |   Reverse Proxy   |                 |      Server       |
                  |                   |  Ignores TE     |                   |
                  +-------------------+                 +-------------------+
                                                                  |
                                                         Processes TE (Stops at 0)
                                                                  |
                                                                  v
                                                        [ Smuggled Bytes Left ]
                                                        [   in TCP Stream     ]
```

#### 1. CL.TE Scenario
* **Front-end:** Processes `Content-Length`, ignores `Transfer-Encoding`.
* **Backend:** Processes `Transfer-Encoding`, ignores `Content-Length`.

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```
* **Execution Flow:** The front-end reads 13 bytes (forwarding everything up to `SMUGGLED`). The backend processes `Transfer-Encoding`, encounters `0\r\n\r\n`, and terminates processing early. The remaining `SMUGGLED` payload stays trapped in the backend's TCP buffer, prepending itself to the next incoming user request.

#### 2. TE.CL Scenario
* **Front-end:** Processes `Transfer-Encoding`.
* **Backend:** Processes `Content-Length`.
* **Execution Flow:** The front-end forwards the request based on the terminating `0\r\n\r\n` chunk. The backend stops reading prematurely based on a short `Content-Length` value, leaving the rest of the request body unread in the pipeline buffer to collide with subsequent requests.

---

## 🎯 Offensive Impact & Exploitation Vectors

1. **Session Hijacking:** Smuggling requests designed to capture incoming user authorization headers or cookies into attacker-accessible endpoints or logs.
2. **Denial of Service (DoS):** Poisoning the TCP connection stream so that legitimate users receive 404/500 errors or administrative redirects instead of their intended resources.
3. **Access Control Bypass:** Forcing the victim's browser to execute privileged administrative actions (`/admin/delete-user`) that the attacker cannot trigger directly due to restricted authorization checks.

---

## 🧪 Practical Scenarios, Questions & Technical Evaluations

### 🔹 Question 1: Standard RFC Compliance Header Handling

* **Question:**
  > If a request containing both `Content-Length: 5` and `Transfer-Encoding: chunked` arrives at an RFC-compliant server, which header will the server process, and which one will it ignore?

* **Student Answer:**
  > A server strictly adhering to standards ignores `Content-Length` entirely and relies on `Transfer-Encoding`.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **100% Accurate Response.**
  * **Technical Detail:** According to **RFC 7230 Section 3.3.3**, when both headers are present, the presence of `Transfer-Encoding` overrides `Content-Length` to prevent ambiguity. The server must strip or ignore `Content-Length` before processing or forwarding the message.

---

### 🔹 Question 2: TCP Stream Poisoning & Response Execution Dynamics

* **Question:**
  > In a CL.TE scenario, why does the smuggled payload remain pending in the connection channel instead of being returned in the attacker's immediate response, and who observes the resulting response?

* **Student Answer:**
  > The victim is the one who will see the result (e.g., the `/admin` page) in their browser!  
  > **Reason:** The attacker poisoned the TCP stream with the pending payload. When the victim sends their request, they reuse the active connection context. The server processes the injected payload concatenated with the victim's request and sends the response back to the victim's browser instead of the page they originally requested.

* **Technical Evaluation & Refinement:**
  * **Verdict:** ✅ **Exemplary & Deeply Accurate Explanation.**
  * **Detailed Mechanics:** Because HTTP/1.1 uses persistent TCP connections (`Keep-Alive`), the backend server treats the leftover smuggled data in the buffer as the beginning of the next request on that socket. When an unsuspecting victim's request arrives on the same pipeline, it appends directly onto the smuggled prefix. Consequently, the backend evaluates the combined request under the victim's session context and routes the output back to the victim's TCP socket connection.
