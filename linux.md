# 🐧 Linux Commands & WSL Setup Guide

## 📌 Part 1: Basic Linux Commands

### 🔹 Navigation

```bash
pwd        # Show current directory
ls         # List files and folders
ls -la     # List all files (including hidden)
cd folder  # Change directory
cd ..      # Move up one directory
```

### 🔹 File & Directory Management

```bash
touch file.txt        # Create file
mkdir folder          # Create directory
rm file.txt           # Delete file
rm -r folder          # Delete folder
cp file1 file2        # Copy file
mv file1 file2        # Move/rename file
```

### 🔹 File Viewing & Editing

```bash
cat file.txt          # Show file content
nano file.txt         # Edit file (simple editor)
vim file.txt          # Advanced editor
head file.txt         # First 10 lines
tail file.txt         # Last 10 lines
```

### 🔹 Permissions

```bash
chmod 755 file        # Change permissions
chown user:file file  # Change ownership
```

### 🔹 System Info

```bash
whoami       # Current user
uname -a     # System info
top          # Running processes
htop         # Better process viewer (if installed)
```

### 🔹 Package Management (Ubuntu/Debian)

```bash
sudo apt update
sudo apt upgrade
sudo apt install package
sudo apt remove package
```

---

## 📌 Part 2: Intermediate Commands

### 🔹 Searching

```bash
grep "text" file.txt
find . -name "file.txt"
```

### 🔹 Networking

```bash
ping google.com
ifconfig
ip a
```

### 🔹 Disk Usage

```bash
df -h
du -sh folder
```

---

## 📌 Part 3: WSL (Windows Subsystem for Linux) Setup

### 🔹 Step 1: Enable WSL

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

### 🔹 Step 2: Restart System

After installation, restart your computer.

### 🔹 Step 3: Choose Linux Distribution

By default, Ubuntu will be installed.

To list available distros:

```powershell
wsl --list --online
```

To install a specific distro:

```powershell
wsl --install -d Ubuntu
```

### 🔹 Step 4: Launch Linux

```powershell
wsl
```

Set username and password when prompted.

### 🔹 Step 5: Update Packages

Inside WSL terminal:

```bash
sudo apt update && sudo apt upgrade
```

---

## 📌 Part 4: Useful WSL Commands

```powershell
wsl --list --verbose     # Show installed distros
wsl --set-default Ubuntu
wsl --shutdown           # Stop WSL
```

---

## 📌 Part 5: Access Windows Files from WSL

```bash
cd /mnt/c   # Access C drive
cd /mnt/d   # Access D drive
```

---

## 📌 Tips

* Use TAB for auto-complete
* Use ↑ key for previous commands
* Use Ctrl + C to stop a process

---

## 🚀 Done!

You now have a basic + intermediate Linux command reference and a complete WSL setup guide.
