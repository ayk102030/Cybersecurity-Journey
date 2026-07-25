# 🐧 Lab 03: Linux File Permissions

In Linux systems, permissions serve as the primary security barrier protecting the system. As a professional hacker or security expert, you must understand exactly how the system interprets these permissions and how you can manipulate them to execute your tools or secure your files.

---

## 🔍 1. How to Read Permission Structure? (Understanding Structure)

When you run the detailed command `ls -l` inside the Terminal, you will notice a string of characters at the beginning of the line that looks like this: 
`-rwxr-xr--`

This string is geometrically divided into **4 main parts** from left to right:

| Part | Characters (Example) | Target Category | Description |
| :--- | :---: | :--- | :--- |
| **First** | `-` or `d` | **File Type** | A dash `-` means a regular file, while the letter `d` means a directory (folder). |
| **Second** | `rwx` | **Owner (User/Owner)** | Permissions of the actual person who created the file. |
| **Third** | `r-x` | **Group** | Permissions of the security group to which the file belongs. |
| **Fourth** | `r--` | **Others** | Permissions of any other user in the system (**The most critical part for a hacker**). |

### 💡 Meanings of Letters and Their Numerical Values:
* **`r` (Read):** Permission to read the file and view its content 👈 **Equals 4**
* **`w` (Write):** Permission to modify or delete the file 👈 **Equals 2**
* **`x` (Execute):** Permission to run the file (if it's a program or an exploit script) 👈 **Equals 1**
* **`-` (None):** Permission is blocked entirely 👈 **Equals 0**

---

## 🔢 2. The Octal Notation System

Professional hackers don't always change permissions using letters; instead, they prefer quick mental math (addition) to combine permission values for each category:

* If you want full permissions (Read + Write + Execute): **4 + 2 + 1 = 7**
* If you want (Read + Write) only: **4 + 2 + 0 = 6**
* If you want (Read + Execute) only: **4 + 0 + 1 = 5**

### 📌 Common Examples in Environments:
* **`777`:** Absolute permissions (Read, Write, Execute) for everyone (Owner, Group, Others). This is a critical security vulnerability if found on sensitive files!
* **`755`:** The owner can do everything (7), while all other system users can only read and execute without modifying (55).

---

## 🛠️ 3. Changing Permissions Command (`chmod`)

When downloading a new exploit tool from platforms like GitHub, Linux automatically blocks it from running for security reasons, giving it read-only permissions. To force the system to make it executable, we use the **`chmod`** (Change Mode) command via two scientific methods:

### 🅰️ Method 1: Symbolic Notation
We use explicit letters to define the category, followed by a control operator, then the permission type:
* `u` (User): Owner
* `g` (Group): Group
* `o` (Others): Others
* `a` (All): Everyone

* **Control Operators:** (`+` to add) , (`-` to strip) , (`=` for exclusive explicit assignment).

* **Practical Examples:**
  * To add execute permission for the owner only:
    ```bash
    chmod u+x notes.txt
    ```
  * To wipe or explicitly set permissions for all categories at once with no extra spaces:
    ```bash
    chmod u=rwx,g=rx,o=x notes.txt
    ```

### 🅱️ Method 2: Octal Notation
The fastest and most professional way because it defines permissions for all three categories (You, Group, Others) in a single 3-digit command with a fixed order: `[Owner] [Group] [Others]`.

* **Exclusive Permission Example (700):**
  If you want to have full permissions while everyone else has absolutely nothing:
  ```bash
  chmod 700 notes.txt
  ```
  *Result when running `ls -l`:* Permissions will appear as `-rwx------` (the `rwx` belongs to you only, the rest is blocked).

* **Blind Execution Permission Example (100):**
  If you want to own execute rights only without reading or modifying, and zeros for the rest:
  ```bash
  chmod 100 notes.txt
  ```
  *Result when running `ls -l`:* Will appear as `--x------`.

---

## 🎯 4. Practical Challenge 03: Breaking Permissions & Testing the Block

To prove how structurally strict Linux is, execute the following steps sequentially inside your Terminal on your `notes.txt` file:

### Step 1: Allocate full file rights to yourself
```bash
chmod 700 notes.txt
ls -l notes.txt
```
> **Note:** You will notice the symbols change to `-rwx------`, meaning technically you are the only one with access.

### Step 2: Strip all permissions (even from yourself!)
Zero out the permissions entirely using:
```bash
chmod 000 notes.txt
```

### Step 3: Attempt to read the file
Try to view the content now:
```bash
cat notes.txt
```
> **Expected Engineering Result:** `cat: notes.txt: Permission denied`

> **💡 Scientific Explanation:** Linux systems are highly strict and do not rely on emotion. When you set permissions to `000` (meaning `---`), the OS instantly applies the block directly to you as the actual owner, denying you read access.

### Step 4: Regain Control
Restore full permissions to the file and read it again to confirm the success of the test:
```bash
chmod 777 notes.txt
cat notes.txt
```

---
🏁 **NOTES:**
* When reading (`ls -l`): We see dashes `-` as empty slots for blocked permissions.
* When writing with letters (`chmod`): We completely ignore dashes and write the continuous characters directly, such as `u=rwx,g=rx,o=x` without any extra spaces.
