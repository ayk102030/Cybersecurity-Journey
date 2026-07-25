# 🐧 Lab 09: Network Forensics with `netstat` & Package Management with `pip`

As a security practitioner, your toolkit must include both deep network visibility tools and package management skills. This lab covers `netstat`—the classic network connection auditor—and `pip`, the Python package manager that serves as both a primary installation vehicle for offensive tools and an entry point for modern supply-chain attacks.

---

## 📡 Part 1: The `netstat` Tool (Network Statistics)

### 🔍 1. Definition & Mechanics
`netstat` stands for **Network Statistics**. It is a foundational utility used to display active network connections, listening ports, routing tables, and interface statistics. 
> **Technical Note:** In modern Linux distributions, `ss` is preferred because it is faster. However, `netstat` remains universally recognizable and vital for legacy systems and cross-platform forensics.

Unlike active network scanners (like `nmap`), `netstat` does not send packets across the wire. Instead, it reads live kernel data directly from the system files located inside the **`/proc/net/`** directory to map out current network states in real-time.

### ⚙️ 2. The Command: `netstat -pnat`
To run a comprehensive forensic sweep, we combine four distinct options:

* **`-p` (Program):** Reveals the **Process ID (PID)** and the exact name of the program managing the socket (e.g., `812/sshd`, `2456/firefox`). *This flag typically requires `root/sudo` privileges to view processes owned by other users.*
* **`-n` (Numeric):** Forces `netstat` to display raw IP addresses and port numbers. This bypasses the **DNS Lookup** process, dramatically increasing the speed of the command output.
* **`-a` (All):** Shows all active connections and listening sockets, capturing states like `LISTEN`, `ESTABLISHED`, `TIME_WAIT`, and more.
* **`-t` (TCP):** Restricts the output specifically to TCP connections.

---

### 📊 3. Key Columns & Connection Output
When reading a `netstat` output grid, a security analyst evaluates five primary data points:

| Column Name | Description | Real-World Example |
| :--- | :--- | :--- |
| **Proto** | The transport layer protocol type. | `tcp`, `udp` |
| **Local Address** | Your local system's IP address and the outbound/inbound port. | `192.168.1.15:22` |
| **Foreign Address** | The IP address and port of the remote machine on the other side. | `104.16.132.229:443` |
| **State** | The socket's current connection status. | `LISTEN`, `ESTABLISHED`, `TIME_WAIT` |
| **PID/Program Name** | The specific process and system ID utilizing the connection. | `812/sshd`, `2456/firefox` |

#### 🛠️ Real-World Output Example:
```text
Proto Local Address      Foreign Address      State         PID/Program Name
tcp   0.0.0.0:22         0.0.0.0:* LISTEN        812/sshd
tcp   192.168.1.15:22    192.168.1.8:50542    ESTABLISHED   812/sshd
tcp   192.168.1.15:443   104.16.132.229:55020 ESTABLISHED   1020/nginx
```

#### 💡 Understanding Essential Connection States:
* **`LISTEN`:** A local service is waiting for new connections. (e.g., `nginx` or `apache` listening for incoming web requests).
* **`ESTABLISHED`:** An active data-exchange pipe is currently open and flowing between your machine and a remote host.
* **`TIME_WAIT`:** The connection has officially closed, but the local OS is keeping the socket open briefly to ensure any stray packets still in transit are gracefully handled before final destruction.

---

### 🛡️ 4. Cybersecurity Applications

#### 🟢 For the Defender (System Auditing)
Defenders use `netstat` to periodically audit baseline network profiles. It allows them to quickly discover unexpected open ports, track active external connections, and tie unauthorized behavior straight back to the parent executable process.

#### 🔴 For Incident Response (Forensics)
During an ongoing active breach investigation, analysts utilize `netstat` to locate anomalous outbound traffic patterns. Finding an unexpected connection does not automatically confirm a compromise, but it serves as an **Indicator of Compromise (IoC)** that guides deeper file and log analysis.

> **⚠️ The Forensic Rule of Context:** > Seeing a live connection bound to a `bash` shell is highly suspicious, but it is not definitive proof of a Reverse Shell. Analysts must evaluate the historical context, execution paths, and parent-child process relationships before declaring a security incident.

---

## 🐍 Part 2: Python Package Management via `pip`

### ⚙️ 1. Technical Mechanics
Modern programming languages rely heavily on pre-built software libraries to handle complex operations (e.g., handling crypto algorithms or making complex HTTP requests). **`pip`** (Pip Installs Packages) is the official management utility that connects directly to the global **Python Package Index (PyPI)** repository to fetch, download, and install these extensions locally.

#### The Standard Syntax:
```bash
pip install [package_name]
```

#### Bulk Installation from GitHub Projects:
When cloning a repository using `git clone`, developers routinely package all necessary software requirements into a text file named `requirements.txt`. Instead of installing dozens of libraries manually, you can instruct `pip` to read the entire file and install everything in a single, automated batch:
```bash
pip install -r requirements.txt
```

---

### 🪓 2. The Cybersecurity Perspective

#### 💥 The Attacker's Prerequisite:
Advanced exploitation frameworks written in Python (such as password-spraying tools or the famous **Impacket** suite used for Active Directory post-exploitation) cannot run without explicit networking and cryptographic dependencies. If an operator cannot confidently use `pip` to satisfy software prerequisites, over 80% of open-source offensive tools will fail immediately, throwing a fatal **`ModuleNotFoundError`**.

#### 🦠 The Defensive Threat: Supply Chain Attacks via Typosquatting
The reliance on package managers has introduced a massive modern threat vector known as **Typosquatting** within the software supply chain.

1. **The Attack:** Malicious actors upload malware to the public PyPI repository and intentionally name it a common misspelling of an incredibly popular library (e.g., uploading a malicious package named `requessts` with an extra 's' to mimic the legitimate `requests` library).
2. **The Compromise:** When an developer or penetration tester makes a typographical error during tool configuration or setup script execution (`pip install requessts`), the system downloads the malicious package. The embedded script executes immediately, granting the attacker Remote Code Execution (RCE) over the target environment.

---

🏁 **Summary for your Security Notebook:**
* `netstat -pnat` extracts active socket connection information straight from `/proc/net/` inside the Linux kernel.
* Correlating **Local/Remote IP, Ports, and PIDs** is essential to determine whether an active link is malicious or authorized.
* `pip` handles essential Python dependencies; however, careful typing is critical to prevent devastating software supply-chain **Typosquatting** compromises.
