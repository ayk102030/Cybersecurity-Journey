# 🌐 Lab 01: Dissecting HTTP Protocol from a Cybersecurity Perspective

Welcome to my first technical write-up! In this lab, I dived deep into the fundamentals of the **HTTP** protocol, analyzing how data travels between the browser and the server. This write-up focuses on security vulnerabilities arising from the misuse of HTTP methods and how to interpret server responses (Status Codes) through the eyes of a **Pentester**.
---

## 🛑 1. The Battle of HTTP Methods: GET vs. POST (A Security Angle)

Many beginner developers choose HTTP methods based solely on code efficiency. However, from a security standpoint, this choice can mean the difference between a secure application and a severe data breach.

### 🔹 GET Requests & The Exposed URL Vulnerability
When using the `GET` method to submit sensitive data (like usernames and passwords), the parameters are appended directly into the URL path, looking something like this: `website.com/login?user=admin&pass=12345`.

* **The Security Threat:** This triggers a well-known vulnerability called **Sensitive Data Exposure in the URL**.
* **Why Attackers Love This Mistake:** URLs are not kept secret; they are logged and stored in **Plain Text** across multiple locations that an attacker can later exploit:
  1. **Browser History:** Anyone with physical access to the device can see the credentials.
  2. **Server Logs:** If an attacker compromises the server, the entire URL history is exposed.
  3. **Network Infrastructure (Proxies/Firewalls):** Intermediate devices log full URLs by default.

*(📸 Reference Images: Screenshot_20260602-140719.jpg & Screenshot_20260601-155346.jpg)*

---

### 🔹 POST Requests & Dispelling the "Encryption" Myth
Unlike GET, the `POST` method removes the data from the URL and places it inside an abstracted section of the request known as the **Request Body**.

*(📸 Reference Image: Screenshot_20260602-140712.jpg)*

* **⚠️ The Dangerous Misconception:** Many believe that a `POST` request is automatically secure just because the credentials aren't visible in the URL bar. 
* **The Hard Reality:** `POST` only protects data from "shoulder surfing" (people looking at your screen) and browser history. The data itself is still transmitted in raw **Plain Text**!
* **Attack Scenario:** If a website uses standard HTTP in a public café, an attacker sniffing the network traffic with a tool like **Wireshark** can easily capture the POST packet and read the password directly from the body.

> **💡 The Golden Rule:** `POST` hides data, but it **does not** encrypt it. True encryption requires the deployment of the secure **HTTPS** protocol.

*(📸 Reference Images: Screenshot_20260602-140705.jpg & Screenshot_20260602-140657.jpg)*

---

## 🧠 2. HTTP Status Codes: A Pentester's Lenses

When a browser sends a request, the server responds with a **3-digit** status code before sending any web content. For a penetration tester, these codes serve as a diagnostic lens to understand exactly what is happening inside the backend infrastructure.

These codes are divided into **5 distinct families** based on their starting digit:

| Code Family | Technical Classification | Cybersecurity Significance | Key Examples |
| **`2xx`** | **Success** | The request was valid, understood, and processed successfully. | `200 OK` (Page exists) <br> `201 Created` (Resource created) |
| **`3xx`** | **Redirection** | The requested resource moved, and the server is redirecting the client. | `301 Moved Permanently` <br> `302 Found` (Temporary redirect) |
| **`4xx`** | **Client Errors** | The fault lies with the user (bad request, wrong syntax, or unauthorized path). | `400 Bad Request` <br> `401 Unauthorized` (Needs login) <br> `403 Forbidden` (No permission) <br> `404 Not Found` <br> `405 Method Not Allowed` <br> `429 Rate Limit` (Too many requests) |
| **`5xx`** | **Server Errors** | The request was valid, but the server crashed or encountered an internal fault. | `500 Internal Server Error` (Code crash) <br> `502 Bad Gateway` <br> `503 Service Unavailable` (Server overload) |

*(📸 Reference Images: Screenshot_20260602-140600.jpg, Screenshot_20260602-140518.jpg, Screenshot_20260602-140430.jpg, Screenshot_20260602-140338.jpg, & Screenshot_20260602-140259.jpg)*

---

## 🎯 Key Takeaways

1. **Secure Development:** Never trust HTTP for sensitive operations. Always combine `POST` methods with strict `HTTPS` implementation for authentication or data submittal points.
2. **Leveraging Status Codes:** Monitoring status codes helps mapping an application's attack surface. For instance, a `500 Internal Server Error` might leak stack traces detailing backend technologies, while a `403 Forbidden` tells a pentester they might have just found a hidden admin panel.
