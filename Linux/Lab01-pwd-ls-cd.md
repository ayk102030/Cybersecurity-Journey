# 🐧 Lab 01: Linux Basics - Navigation & File Exploration

## 📍 1. The `pwd` Command (Where am I?)
When you open the Terminal, you are placed inside a specific directory, but you cannot visually see it.

* **Command:** `pwd` (Print Working Directory)
* **Purpose:** It outputs the full path of the directory you are currently standing in.
* **Expected Output:** You will see something like `/home/kali` (this is called your user's home directory).

---

## 📂 2. The `ls` Command (What is inside here?)
Used to know what files or other folders are inside the directory you are currently standing in.

* **Command:** `ls` (List)
* **The Hacker's Approach (`ls -la`):** Typing just `ls` shows only standard files. However, a hacker always uses `ls -la`.
  * `-l`: Displays files in a detailed list showing file size and permissions (we will learn them soon).
  * `-a`: Displays hidden files (which start with a dot `.`). These are the files where developers often hide passwords and vulnerabilities!

---

## 🚶‍♂️ 3. The `cd` Command (Moving Around)
Used to enter a specific folder or exit from it.

* **Command:** `cd` (Change Directory)
* **How to use it:**
  * To enter a directory named Downloads: `cd Downloads`
  * To go back one step (to the previous directory): `cd ..` (two dots)

---

## 🛠️ Practical Challenge (Completed)
Successfully executed the following workflow in the Kali Linux terminal:
1. Typed `pwd` to see the current location.
2. Typed `ls -la` to see hidden files and details.
3. Entered the Desktop folder by typing `cd Desktop` (ensuring the capital 'D').
4. Typed `pwd` again to confirm that the path has successfully changed.
