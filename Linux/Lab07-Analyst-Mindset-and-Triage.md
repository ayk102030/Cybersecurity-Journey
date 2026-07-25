# 🐧 Lab 07: The Security Analyst Mindset - Identifying Malicious Connections

In reality, you cannot definitively classify a network connection as malicious simply because you see the `bash` process or a specific port being used. Multiple indicators must be gathered and correlated before reaching a conclusion. 

Here is exactly how a professional Security Analyst thinks and investigates:

---

## 🔍 1. The Program Name (Process)
This is the very first thing we look at.

### ✅ Normal & Expected
If you see well-known applications connecting to the internet, such as `firefox`, `chrome`, `curl`, `apt`, `ssh`, or `discord`, this is usually expected behavior.
> **Example:** `ESTAB ... users:(("firefox",pid=1234))`
> **Analysis:** The program is a web browser. The user opened a website. This is a normal connection.

### ⚠️ Requires Investigation
If you see processes like `bash`, `python3`, or `sh` establishing connections, you must ask: *"Why is a command-line shell opening a network connection?"*
Usually, `bash` does not connect to the internet on its own. However, seeing it is an *indicator*, not definitive proof of a hack. 
> **Legitimate Example:** You might have run a command like `curl https://example.com | bash` or an administrative script. 

---

## 📊 2. The Connection State (`State`)

* **`LISTEN`:** The program is actively waiting for incoming connections. 
  * *Example:* `sshd LISTEN 22` (This is completely normal if you run an SSH server).
* **`ESTAB` (Established):** There is a live, ongoing connection between your machine and a remote device. This is where active analysis begins.

---

## 🚪 3. The Port Number
Every service uses a specific port to communicate.

| Port | Protocol / Service | Typical Use Case |
| :--- | :--- | :--- |
| **80** | HTTP | Unencrypted web browsing |
| **443** | HTTPS | Secure web browsing |
| **22** | SSH | Remote administration |
| **53** | DNS | Domain name resolution |
| **25** | SMTP | Email routing |

If you see an outbound connection to port `443`, it is highly likely to be standard web traffic. 

**What about ports like 4444 or 5555?**
Many penetration testing tools (like Metasploit) use these ports by default. However, seeing port `4444` alone does not mean you are hacked. It could be a script you wrote, a training lab, or an internal service. The port alone is never enough to pass judgment.

---

## 🌍 4. The Remote IP (`Peer Address`)
This is one of the most critical pieces of evidence.

* **Known IP (e.g., `142.250.x.x`):** Might belong to Google or another known, legitimate service.
* **Unknown IP (e.g., `198.51.100.x`):** An unrecognized address. 
  
As an analyst, you ask:
* *Do I know this server?*
* *Is this connection expected?*
* *Does this IP belong to a service I actually use?*
If the answer is no, the investigation escalates.

---

## 🤔 5. Is the Connection Expected?
This is the single most important question in cybersecurity triage.

* **Expected:** You open `firefox`, and an internet connection appears. This makes logical sense.
* **Unexpected:** You have no applications open, yet `python3` reaches out to an external IP every 60 seconds. This is highly abnormal and warrants immediate investigation.

---

## 🛠️ 6. Practical Triage Examples

### Case 1: Normal Activity
`ESTAB 10.0.2.15:50120  34.160.xxx.xxx:443  users:(("firefox",pid=3251))`
* **Analysis:** The program is Firefox. The port is 443. The user is securely browsing the web. 
* **Verdict:** Normal.

### Case 2: Suspicious Activity
`ESTAB 10.0.2.15:39220  203.0.113.8:4444  users:(("bash",pid=4120))`
* **Analyst Questions:** Why is `bash` connecting to the network? Why port `4444`? Did I initiate this? What is this strange IP? Who started this process?
* **Verdict:** If no legitimate explanation is found, this is highly indicative of a **Reverse Shell** or malicious activity. Further forensic evidence is required (checking command history, identifying the parent process, etc.).

---

## 🏆 7. The Security Analyst's Golden Rule

Never rely on a single indicator of compromise (IoC) like `bash`, port `4444`, or an unfamiliar IP. Instead, correlate multiple indicators together:

* ✅ What is the program?
* ✅ Where is it connecting?
* ✅ On what port?
* ✅ Is this connection expected?
* ✅ Who initiated the process?
* ✅ Are there other supporting indicators?

The more these suspicious indicators align, the higher the probability of a genuine threat. **In cybersecurity, context is everything!**
