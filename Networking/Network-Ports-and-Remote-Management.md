# 🔍 Lab 06: Network Ports, Remote Management & File Transfer Protocols

Welcome to the sixth lab! In this module, we will explore how we interact with remote servers across the internet. We will demystify network ports, dive into Remote Management protocols like SSH and Telnet, understand how files are transferred via FTP, and analyze the critical cybersecurity vulnerabilities associated with each.

---

## 🚪 1. What are Ports and Where to Find Them?

Scientifically, **Ports** are not visible to the naked eye; rather, they are "virtual doors" located within a server's operating system (like Linux servers, cloud servers such as AWS and Azure, or even networking devices like routers). Every service running on a server requires a specific port to receive and send data through it.

### 🕵️‍♂️ The Hacker's Approach: Port Scanning
In the cybersecurity realm, a hacker or a security engineer discovers these ports through a phase known as **Scanning**:
* A tool like **Nmap** (the standard and most famous tool in cybersecurity) is utilized.
* The tool sends data packets to the target server's IP address, testing ports from `1` to `65535`.
* **The Result:** A report is generated showing the open ports and the specific services running on them.

> **Example Nmap Output:**
> `22/tcp   open   ssh`
> `21/tcp   open   ftp`
> `80/tcp   open   http`

---

## 💻 2. Remote Server Management: SSH vs. Telnet

**Remote Management** means that the server might be physically located in the US while you are sitting in your room, yet you can fully control it (manage files, install software, stop websites) via the **Command Line**.

### ⚙️ The Scientific Mechanism
When port `22` or `23` is open, there is a program inside the server acting as a "listener" (**Daemon**). From your machine, you open an application like **Terminal** (in Linux/macOS) or **PuTTY** (in Windows) and type the connection command targeting the server's IP and port. 

Once you enter the correct username and password, your screen essentially becomes the server's screen, and any command you type is immediately executed on the server's processor.

### 🔴 Telnet (Port 23) - The Insecure Way
* **Legacy Protocol:** Historically used for remote server control.
* **The Security Flaw:** It transmits data and text in completely **Plain Text**. This means that anyone eavesdropping on the network can clearly read the username and password.
* **Connection Example:** `telnet 192.168.1.10`
* **Cybersecurity Perspective:** If you run an Nmap scan and find `23/tcp open telnet`, this is a massive red flag and a severe security vulnerability that requires immediate evaluation.

### 🟢 SSH (Secure Shell) - Port 22 - The Secure Way
* **Modern & Secure Protocol:** The encrypted and secure alternative to Telnet.
* **Connection Example:** `ssh user@192.168.1.10`
* **Why is it secure?** All data (including passwords) is sent over a fully **Encrypted** connection. If someone tries to spy on the network traffic, they will only intercept incomprehensible gibberish.
* **Cybersecurity Perspective:** Finding port `22/tcp open ssh` is completely normal and standard practice on servers for secure administration.

---

## 📁 3. File Transfer Protocol (FTP)

The **FTP** protocol is dedicated solely to transferring files (it does not provide a remote command-line interface). Its primary purpose is to upload files from your local machine to the server or download them.

* **Ports Used:** It typically operates on Port `21` (**Control Channel**) and Port `20` (**Data Channel**).
* **Practical Example:** A web developer uses an FTP client like *FileZilla* to upload website assets (`index.html`, `style.css`, `logo.png`) to the hosting server.

### ⚠️ The Security Flaws of Traditional FTP
Just like Telnet, the traditional FTP protocol is unencrypted and transmits data in **Plain Text**. If you log in, your credentials (`Username: admin` / `Password: 123456`) will travel across the network completely exposed.
* **Secure Alternatives:** Traditional FTP is widely being replaced by **SFTP** (which relies on SSH encryption over Port 22) or **FTPS** (which adds SSL/TLS encryption to standard FTP).

### 🏴‍☠️ The Cybersecurity Perspective on FTP
If you discover `21/tcp open ftp` during an Nmap scan, a penetration tester must ask the following questions:
1. **Does it allow Anonymous Login?** This is a critical vulnerability allowing anyone in the world to access the server without needing a username or password.
2. **Is the service outdated and does it contain known vulnerabilities (CVEs)?**
3. **Can files be uploaded?** If "Anonymous Login" is enabled AND file uploads are permitted, an attacker could easily upload a malicious payload (like a **Web Shell**) and take full control of the server.
4. **Is encryption being utilized or not?**

---

## 📊 4. Protocol Summary & Security Status

| Service | Purpose | Port | Security Status |
|---|---|---|---|
| **HTTP** | Web Browsing | `80` | Plain Text ❌ |
| **HTTPS** | Secure Web Browsing | `443` | Encrypted ✅ |
| **Telnet** | Remote Management | `23` | Plain Text ❌ |
| **SSH** | Secure Remote Management | `22` | Encrypted ✅ |
| **FTP** | File Transfer | `21` / `20` | Plain Text ❌ |
| **SFTP** | Secure File Transfer | `22` | Encrypted ✅ |
