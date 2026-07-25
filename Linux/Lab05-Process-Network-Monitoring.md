# 🐧 Lab 05: Process Management & Network Monitoring

In cybersecurity, whether you are attacking a target or defending your own system, you must be able to answer one critical question: *"What is running in the background right now?"* 

Hackers often hide their tracks by running malicious scripts (like reverse shells) deep in the system's background. In this lab, we dissect the tools used to act as a system radar to uncover and terminate these hidden threats, and we learn how to read network states like a forensic investigator.

---

## 🔬 1. Uncovering Running Processes (`ps aux`)

The `ps` (Process Status) command is the primary tool for listing active processes. However, running it alone only shows your current terminal's processes. To perform a deep system sweep, we combine it with three powerful flags: **`aux`**.

### 🛠️ Dissecting the `aux` Flags:
* **`a` (All):** Displays processes belonging to *all* users on the system (including `root`).
* **`u` (User-oriented):** Expands the output to show detailed metrics, including which user started the program, its Process ID (PID), and its CPU/RAM consumption.
* **`x` (eXclude terminal):** **The most critical security flag!** It forces the system to reveal processes running in the "dark background" that are completely detached from any visible terminal screen. Professional hackers hide their malware here. With this flag, the system pulls them out of the shadows (their terminal `TTY` status will show up as a `?`).

---

## 📡 2. The Network Radar (`ss -antp`)

To detect if a hidden process is communicating with the outside world, we use `ss` (Socket Statistics). We use the `-antp` flags to show **A**ll sockets, **N**umeric IPs/Ports, **T**CP protocol only, and the responsible **P**rocess.

### 📊 Forensic Breakdown of the `ss` Output Columns:
When you run this command, you must understand exactly how to read the grid:

1. **State:** The current status of the connection.
   * `LISTEN`: A port is wide open on your machine, quietly "listening" and waiting for an external device to connect. *(If this port is unknown, it could be a hacker's Backdoor!)*
   * `ESTABLISHED`: A live, active, two-way connection is currently happening between your machine and another device in the world.
2. **Recv-Q & Send-Q (Queues):** The amount of data waiting to be received or sent. 
   * *Normal:* Should be `0` or tiny numbers that vanish quickly.
   * *Danger:* If you see massive, frozen numbers here, it indicates a Denial of Service (DoS) attack or massive **Data Exfiltration** (a hacker downloading your files in the background).
3. **Local Address:Port:** Your machine's IP and the port it is using.
   * `127.0.0.1:80`: Open internally only. No one outside can see it.
   * `0.0.0.0:22` or `*:22`: Open to the entire world! Anyone can attempt to connect.
4. **Peer Address:Port:** The remote target (The other side of the connection).
   * If `LISTEN`, this is empty `*` (waiting for anyone).
   * If `ESTABLISHED`, this reveals the exact IP of the server or victim you are connected to.
5. **Process:** The "culprit" or program that opened the connection. It shows the program name and its PID (e.g., `users:(("ssh",pid=1024,fd=3))`).

---

## ⚔️ 3. Terminating Threats (`kill`)

Once you identify an active threat via its PID, you must neutralize it.
* **Command:** `kill -9 <PID_NUMBER>`
* **Why `-9`?** It is the absolute termination signal (`SIGKILL`). It tells the Linux kernel to instantly and unconditionally kill the process without waiting for it to close gracefully.

---

## 🎯 4. Practical Challenge 05: The Activation Challenge

If you run `sudo ss -antp` on a fresh Kali Linux machine, the table will likely be entirely empty! 
> **Technical Reason:** Modern Kali is designed to be highly secure. It does not start any background services (like web or SSH servers) automatically. If you haven't opened a browser, there are no `ESTABLISHED` connections and no `LISTEN` ports.

As a hacker, you must know how to "provoke" the system to generate data. Let's force the radar to light up:

### 🧪 Experiment 1: Generating an Outbound Connection (`ESTABLISHED`)
1. Open the Firefox browser inside Kali and visit a website (e.g., `google.com`).
2. Leave the browser open, return to your Terminal, and run the radar:
```bash
   sudo ss -antp
   ```
> **Observation:** You will now see a flood of lines with the state `ESTABLISHED` utilizing port `443` (HTTPS), proving your browser is actively talking to external servers.

### 🧪 Experiment 2: Opening a Listening Gateway (`LISTEN`)
Let's force Kali to open a port and wait for connections by starting the built-in SSH remote-control server.
1. Start the SSH service:
```bash
   sudo systemctl start ssh
   ```
2. Run the radar again:
```bash
   sudo ss -antp
   ```
> **Observation:** You will see a brand new line that wasn't there before. The state is `LISTEN`, the local address shows `0.0.0.0:22` (Port 22 is SSH), and it is actively waiting for an incoming connection!

✅ **Mission Accomplished! You now know how to map processes, monitor network sockets, and read system states like a true forensic analyst.**
