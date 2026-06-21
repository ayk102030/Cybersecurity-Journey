# 🔒 Lab 03: Demystifying HTTPS, SSL/TLS, and Cryptographic Handshakes

Welcome to the third lab of my cybersecurity journey! After analyzing how raw HTTP transmits data in clear text, this lab dives deep into **HTTPS (Hypertext Transfer Protocol Secure)**. We will explore how encryption protocols protect data in transit, how trust is established on the web, and how attackers attempt to break this secure layer.

---

## 🆚 1. HTTP vs. HTTPS: The Core Security Difference

The difference between HTTP and HTTPS can be summarized in one critical concept: **Data Confidentiality**. 

* **HTTP (Unsecure):** Data (passwords, session tokens, financial info) travels across the network as **Plaintext**. Anyone sitting in the path of the traffic (like an attacker on public Wi-Fi) can read it effortlessly.
* **HTTPS (Secure):** Data is transformed into unreadable **Ciphertext** using **SSL/TLS** before it leaves the device. Even if intercepted, it looks like random gibberish to an attacker.

---

## 🔑 2. The Cryptographic Backbone: Symmetric vs. Asymmetric Encryption

HTTPS combines two distinct cryptographic methods to solve the ultimate dilemma: *How do two devices securely share a secret key over a public, insecure network?*

### 🔄 A. Symmetric Encryption (The Speed Demon)
* **How it works:** It uses a single, identical **Shared Key** to both encrypt and decrypt data.
* **Pros:** Extremely fast and highly efficient for transferring large amounts of web traffic.
* **The Vulnerability:** If you send this key over the internet to the server, an attacker can intercept it and decrypt all subsequent traffic.

### 🔏 B. Asymmetric Encryption (The Door Lock)
* **How it works:** It uses a mathematically linked **Key Pair**:
  * **Public Key:** Shared openly with the world; anyone can use it to *encrypt* data.
  * **Private Key:** Kept strictly secret by the server; it is the *only* key capable of *decrypting* data that was encrypted by its corresponding Public Key.
* **Pros:** Solves the key exchange problem completely because the Private Key never travels over the network.
* **Cons:** Computationally heavy and slow compared to Symmetric Encryption.

> **💡 How HTTPS Combines Both:** HTTPS uses **Asymmetric Encryption** at the very beginning to securely negotiate and exchange a temporary **Symmetric Key**, which is then used to encrypt the actual web session data at blazing fast speeds.

---

## 🤝 3. The TLS Handshake: Establishing the Secure Channel

Before a single byte of a web page is loaded, the browser and server execute a precise cryptographic dance known as the **TLS Handshake**. This process achieves mutual negotiation and key agreement.

### 📋 Key Handshake Steps:
1. **ClientHello & ServerHello:** The browser and server agree on the version of TLS and the cryptographic algorithms (**Cipher Suites**) they will use.
2. **Certificate Exchange:** The server sends its **Digital Certificate** (containing its Public Key) to the browser.
3. **Key Generation:** The browser verifies the certificate, generates a random session key, encrypts it with the server's Public Key, and sends it back. Only the server can decrypt this using its private key.
4. **Session Swapping:** Both sides switch to Symmetric Encryption using this new shared key for the rest of the session.

---

## 📜 4. Digital Certificates & Certificate Authorities (CA)

How does your browser know that `google.com` is actually Google and not an attacker's rogue server? This is where **Digital Certificates** come in.

* **The Role of a CA:** A Certificate Authority (like Let's Encrypt or DigiCert) acts as a trusted third-party notary. They verify the identity of a website owner and digitally sign their SSL/TLS certificate.
* **Browser Trust Store:** Operating systems and browsers come pre-installed with a list of trusted Root CAs. If a server provides a certificate signed by an unknown or untrusted CA, the browser throws a massive security warning screen to protect the user.

---

## 🏴‍☠️ 5. The Pentester’s Perspective: Breaking HTTPS via SSL Stripping

From a penetration testing standpoint, attacking modern TLS encryption directly is virtually impossible due to its mathematical strength. Instead, attackers target the **implementation** and human behavior using a critical attack technique called **SSL Stripping (Downgrade Attack)**.

### 🔴 The SSL Stripping Attack Vector
Most users don't type `https://` manually; they type `website.com`, which initially triggers an unencrypted HTTP request. The server then responds by redirecting them to the secure HTTPS version.

1. An attacker places themselves in the middle of the network (**Man-in-the-Middle**).
2. When the user requests the site via HTTP, the attacker intercepts it, establishes a secure HTTPS connection with the real server on behalf of the user, but serves a standard, unencrypted **HTTP** version back to the victim.
3. The victim continues browsing an unencrypted site, completely unaware that the attacker is capturing all passwords and session tokens passing through.

### 🛡️ Defensive Remediation: HSTS
To mitigate downgrade attacks, security professionals deploy **HSTS (HTTP Strict Transport Security)** headers. This forces the browser to *never* communicate with the site over HTTP, ensuring all connections are natively upgraded to HTTPS locally before hitting the network.

