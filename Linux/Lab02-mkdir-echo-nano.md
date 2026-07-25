# 🐧 Lab 02: Linux File Management & Text Editing

## 📁 1. Creating a New Folder (`mkdir`)
* **Command:** `mkdir` (Stands for **Make Directory**)
* **How to use it:** Type the command followed by the name you want to give to the folder.
* **Example:** `mkdir Target_Company` (This will create a new directory on your desktop with this name).

---

## 📝 2. Creating a File and Writing Quickly Inside It (`echo` with the arrow `>`)
* **Mechanism:** The `echo` command prints text, and the arrow symbol `>` takes this text and redirects it into a file (this process is called **Redirection**).
* **Example:**
```bash
echo "IP: 192.168.1.50" > info.txt
```
* **Result:** This command will instantly create a new text file named `info.txt` and place the text written inside the quotes directly into it.

---

## 🔍 3. Reading File Contents (`cat`)
* **Command:** `cat` (Stands for **Concatenate**)
* **Security Purpose:** Reviewing what is inside a text file and reading it fully on the screen without needing to open any additional programs.
* **Example:** `cat info.txt` (This will display the IP address we wrote inside the file).

---

## 🛠️ Your Second Practical Challenge (Prove to yourself that you control the system)
Since you are still standing inside the `Desktop` directory in your Terminal, execute these steps sequentially:

1. Create a new folder named `Pentest` (using `mkdir`).
2. Enter this new folder (using `cd`).
3. Create a file inside it named `notes.txt` and write any sentence of your choice inside it (using `echo` and the arrow `>`).
4. Read the contents of the file to confirm the text was written successfully (using `cat`).

> **Note:** Execute this challenge now on your Kali machine. If you look at your desktop, you will see the folder and files appearing and changing in real-time based on the commands you write with your own hands!

---

## 🚀 4. The Professional Way: Free Editing Using (`nano`)
The `echo` command is excellent for quick single-line writing. However, as a hacker, when you want to edit a file containing many lines or write a complex script, you will not use `echo`. Instead, you will use an interactive text editor inside the Terminal called `nano`.

* **Command:**
```bash
nano notes.txt
```
* **What will happen?** The black terminal screen will turn into an interactive text editor (similar to Notepad). You can move around using the arrow keys, write, delete, and edit with total freedom.

### 💾 How to Save and Exit from the Nano Editor:
1. After you finish writing, press **`Ctrl + O`** (then press **`Enter`**) to save your modifications.
2. Press **`Ctrl + X`** to close the editor and return to the standard command screen.

---

## 🛠️ Try it Yourself Now:
* Write a command using `>>` to append a new line to your current file, and then read it using `cat`.
* Open the file using `nano notes.txt` and manually write two lines, then save and exit (**`Ctrl + O`** then **`Enter`** then **`Ctrl + X`**).
