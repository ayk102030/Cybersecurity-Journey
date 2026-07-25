# 🐧 Lab 10: Environment Variables & Text Extraction with `cut`

In Linux administration and cybersecurity operations, efficiency and speed are critical. This lab introduces terminal **Variables** for dynamically storing target data, and the **`cut`** utility—a powerful tool for carving out specific data blocks and columns from massive log files and system configurations.

---

## 💾 1. Understanding Terminal Variables

A variable in Linux is a named storage space inside your active terminal session. It allows you to save a value (such as a target IP address) so you do not have to manually retype it into every subsequent command.

### ⚠️ The Engineering Rule: Zero Spaces Allowed!
When declaring a variable, you **must not** place spaces before or after the equal sign (`=`). 
```bash
# ❌ INCORRECT (Will throw a command execution error)
IP = "10.10.10.5"

# ✅ CORRECT
IP="10.10.10.5"
```
> **Technical Reason:** If you include spaces, the Linux shell misinterprets the variable name (`IP`) as an independent system command and attempts to execute it as a program.

### 📡 Calling the Variable
To retrieve or execute the value stored inside your variable, prefix its name with a dollar sign (**`$`**):
```bash
ping $IP
```

### 🛡️ Cybersecurity Use Case:
When initiating a security assessment against a target server, the very first tactical step is storing the target's IP address inside an environment variable. You can then pass `$IP` directly into automation scripts or tools like `nmap`, `gobuster`, or `nikto` seamlessly.

---

## 🪓 2. The Text Slicing Tool (`cut`)

Imagine reading a raw database dump or system log file containing excessively long lines of text, but you only need a single column (like usernames or ports). The `cut` command allows you to slice off the noise and extract exactly what you want.

### 🛠️ Mode 1: Character Positioning (`-c`)
The `-c` flag extracts characters based strictly on their exact visual position within a line, ignoring words or symbols.

* **Extract the first character only:**
  ```bash
  cut -c 1 file.txt
  ```
* **Extract a fixed range (Characters 1 through 5):**
  ```bash
  cut -c 1-5 file.txt
  ```
* **Extract from character 3 to the absolute end of the line:**
  ```bash
  cut -c 3- file.txt
  ```
* **Extract from the beginning of the line up to character 4:**
  ```bash
  cut -c -4 file.txt
  ```
* **Cherry-pick specific coordinates only (Characters 1, 3, and 5):**
  ```bash
  cut -c 1,3,5 file.txt
  ```

---

### 🧱 Mode 2: Field & Delimiter Extraction (`-f` & `-d`)
When analyzing structured text files, data is typically split into clean columns using a recurring character (such as a space, a colon, or a comma). This separating symbol is called a **Delimiter**.

* **`-d` (Delimiter):** Specifies the symbol acting as the boundary between columns.
* **`-f` (Field):** Specifies the column number you want to slice out and display.

#### 📝 Visual Architecture of Fields:
Imagine a data string formatted like this: `Ali:20:Baghdad`. It is split into 3 fields using a colon (`:`) delimiter:

```text
   Field 1         Field 2         Field 3
  [  Ali  ]   :   [  20   ]   :   [ Baghdad ]
      |               |               |
   Username          Age           Location
```

* **To extract Field 2 (Age) using a colon delimiter:**
  ```bash
  cut -d ":" -f 2 file.txt
  ```
* **Adapting to other delimiters (Commas or Semicolons):**
  ```bash
  cut -d "," -f 1 data.csv
  cut -d ";" -f 3 data.log
  ```

---

## 🎯 3. Practical Forensic Challenge: Harvesting System Users

Let's apply this directly to a sensitive system file. The `/etc/passwd` file stores basic user configurations on all Linux operating systems. The fields within this file are tightly packed and separated exclusively by colons (`:`).

Execute this precise pipeline command inside your Kali Linux terminal to strip away the system metadata and isolate a clean list of all local accounts:

```bash
cat /etc/passwd | cut -d ":" -f 1
```

### 🔬 Forensic Breakdown of the Chain:
1. **`cat /etc/passwd`**: Reads the entire user configuration database and drops it into the pipeline.
2. **`|` (The Pipe)**: Intercepts that raw textual data stream and passes it directly as input to the next tool.
3. **`cut -d ":"`**: Configures the parsing constraints, defining the colon (`:`) as our delimiter.
4. **`-f 1`**: Instructs the tool to drop fields 2 through 7, printing *only* Field 1—which exclusively contains the system usernames.

---

🏁 **Summary for your Security Notebook:**
* Use `NAME="value"` to define environment variables for quick reference via `$NAME`.
* Use `cut -c` when you need to parse data based on fixed string lengths or positions.
* Use `cut -d [symbol] -f [number]` to easily slice specific columns out of structured logs, CSVs, or system configuration dumps.
