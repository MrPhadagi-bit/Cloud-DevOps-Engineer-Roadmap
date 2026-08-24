#  chmod and File Permissions

> **GitHub Lab** | Based on [Linux for Programmers #4 | chmod and File Permissions]

## Introduction

Every file and directory in Linux has an **owner**, a **group**, and **permission access rights** for three different classes of users. Understanding and managing these permissions is essential for system security and collaboration.

This lab will teach you how to:
- View file permissions using `ls -l`
- Understand the permission structure
- Modify permissions using `chmod` (symbolic and numeric modes)
- Apply permissions recursively
- Handle special permission scenarios

---

## What is `chmod`?

`chmod` stands for **"change mode"** — it modifies the read, write, and execute permissions of files and directories.

### The Three Permission Types

| Permission | Letter | Meaning on Files | Meaning on Directories |
|------------|--------|------------------|------------------------|
| **Read**   | `r`    | View file contents | List directory contents (`ls`) |
| **Write**  | `w`    | Modify file contents | Create/delete files inside |
| **Execute**| `x`    | Run as a program/script | Enter the directory (`cd`) |

### The Three User Classes

| Class | Letter | Description |
|-------|--------|-------------|
| **Owner** | `u` | The user who owns the file |
| **Group** | `g` | Users in the file's group |
| **Others**| `o` | Everyone else on the system |
| **All**   | `a` | All users (`u` + `g` + `o`) |

---

## Understanding `ls -l` Output

Before changing permissions, you need to know how to **read** them.

### Step 1: Run the command

```bash
ls -l filename.txt
```

### Step 2: Analyze the output

```
-rw-r--r-- 1 owner group 12288 Apr  8 20:51 filename.txt
|[-][-][-]-  [------] [---]
| |  |  | |     |       |
| |  |  | |     |       +------------> Group name
| |  |  | |     +--------------------> Owner name
| |  |  | +--------------------------> Alternate Access Method
| |  |  +----------------------------> Others Permissions
| |  +-------------------------------> Group Permissions
| +----------------------------------> Owner Permissions
+------------------------------------> File Type
```

### Step 3: Decode the permission string

```
-rw-r--r--
│└┬┘└┬┘└┬┘
│ │  │  │
│ │  │  └── Others:  r--  (read only)
│ │  └───── Group:   r--  (read only)
│ └──────── Owner:   rw-  (read + write)
└────────── File type: - = regular file
```

### File Type Characters

| Character | Type |
|-------------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |

### Example Breakdown

```bash
$ ls -l
-rw-r--r-- 1 alice devs  2048 Aug 24 10:00 script.sh
drwxr-xr-x 2 alice devs  4096 Aug 24 09:30 projects/
```

| File | Type | Owner | Group | Others | Interpretation |
|------|------|-------|-------|--------|----------------|
| `script.sh` | `-` (file) | `rw-` | `r--` | `r--` | Alice can read/write; group & others can only read |
| `projects/` | `d` (directory) | `rwx` | `r-x` | `r-x` | Alice has full control; group & others can list and enter |

---

## The `chmod` Command

### Basic Syntax

```bash
chmod [OPTIONS] MODE FILE...
```

### Common Options

| Option | Description |
|--------|-------------|
| `-R` | Recursive — apply to all files and subdirectories |
| `-v` | Verbose — print a message for each file processed |
| `-c` | Changes — report only when a change is actually made |
| `--reference=RFILE` | Copy permissions from a reference file |

---

## Symbolic vs Numeric Notation

### 🔤 Symbolic (Text) Method

Uses letters and operators to modify permissions.

#### Syntax

```bash
chmod [WHO][OPERATOR][PERMISSIONS] FILE
```

#### Operators

| Operator | Action |
|----------|--------|
| `+` | **Add** the permission |
| `-` | **Remove** the permission |
| `=` | **Set exactly** to these permissions (overwrites existing) |

#### Examples

```bash
# Add execute permission for the owner
chmod u+x script.sh

# Remove write permission for group and others
chmod go-w secret.txt

# Set group permissions to exactly read and execute
chmod g=rx shared_folder/

# Add read permission for everyone
chmod a+r document.txt
# (same as: chmod +r document.txt)

# Remove all permissions for others
chmod o= private.key

# Multiple changes in one command
chmod u+rwx,go=rx,g-w myfile
```

### 🔢 Numeric (Octal) Method

Uses a 3-digit number where each digit represents permissions for owner, group, and others.

#### Permission Values

| Permission | Value |
|------------|-------|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |
| No permission | 0 |

#### Calculating Permissions

| Desired | Calculation | Digit |
|---------|-------------|-------|
| `rwx` | 4 + 2 + 1 | **7** |
| `rw-` | 4 + 2 + 0 | **6** |
| `r-x` | 4 + 0 + 1 | **5** |
| `r--` | 4 + 0 + 0 | **4** |
| `-wx` | 0 + 2 + 1 | **3** |
| `-w-` | 0 + 2 + 0 | **2** |
| `--x` | 0 + 0 + 1 | **1** |
| `---` | 0 + 0 + 0 | **0** |

#### Examples

```bash
# Owner: rwx, Group: r-x, Others: r-x
chmod 755 script.sh

# Owner: rw-, Group: r--, Others: r--
chmod 644 document.txt

# Owner: rwx, Group: ---, Others: ---
chmod 700 private.key

# Owner: rw-, Group: ---, Others: ---
chmod 600 ~/.ssh/id_rsa
```

### 📊 Common Numeric Values Cheat Sheet

| Value | Permissions | Use Case |
|-------|-------------|----------|
| `777` | `rwxrwxrwx` | Full access for everyone ⚠️ (avoid) |
| `755` | `rwxr-xr-x` | Executable scripts, directories |
| `750` | `rwxr-x---` | Private executable, group can run |
| `700` | `rwx------` | Private scripts, SSH keys directory |
| `644` | `rw-r--r--` | Standard files, web content |
| `640` | `rw-r-----` | Semi-private files |
| `600` | `rw-------` | Private keys, password files |
| `444` | `r--r--r--` | Read-only files |
| `000` | `---------` | No access |

---

## Practical Examples

### Example 1: Making a Script Executable

```bash
# Create a script
echo '#!/bin/bash' > hello.sh
echo 'echo "Hello, World!"' >> hello.sh

# Check current permissions
ls -l hello.sh
# Output: -rw-r--r-- 1 user group ... hello.sh

# Add execute permission
chmod u+x hello.sh

# Verify
ls -l hello.sh
# Output: -rwxr--r-- 1 user group ... hello.sh

# Run it
./hello.sh
```

### Example 2: Securing a Private Key

```bash
# SSH private keys should be restricted
chmod 600 ~/.ssh/id_rsa

# Verify
ls -l ~/.ssh/id_rsa
# Output: -rw------- 1 user group ... id_rsa
```

### Example 3: Setting Up a Web Directory

```bash
# Create a website directory
mkdir /var/www/my_site

# Set directory permissions (owner full, others read+execute)
chmod 755 /var/www/my_site

# Set file permissions (owner full, others read only)
chmod 644 /var/www/my_site/index.html
```

### Example 4: Recursive Permission Changes

```bash
# Apply permissions to a directory and everything inside
chmod -R 755 my_project/

# ⚠️ Be careful! Test first:
chmod -R 755 my_project/  # This affects ALL files and subdirs
```

### Example 5: Copying Permissions from Another File

```bash
# Make file2 have the same permissions as file1
chmod --reference=file1.txt file2.txt
```

### Example 6: Bulk File vs Directory Permissions

```bash
# Set all directories to 755
find /var/www -type d -exec chmod 755 {} \;

# Set all files to 644
find /var/www -type f -exec chmod 644 {} \;
```

---

## Special Cases

### What Happens When You Don't Specify a User?

If you omit the user class in symbolic mode, `chmod` applies to **all users** (`a`), but respects the `umask` setting.

```bash
# This adds execute for all (owner, group, others)
chmod +x script.sh

# Same as:
chmod a+x script.sh
```

### Setting Multiple Permissions at Once

```bash
# Add read and write for owner, read for group, nothing for others
chmod u+rw,g+r,o= file.txt

# Set exact permissions for all classes
chmod u=rwx,g=rx,o=r file.txt
```

### Special Permission Bits (Advanced)

| Bit | Numeric | Symbolic | Description |
|-----|---------|----------|-------------|
| **SUID** | `4000` | `u+s` | Run file as the owner |
| **SGID** | `2000` | `g+s` | Run file as the group / inherit group in directories |
| **Sticky** | `1000` | `+t` | Only owner can delete files in directory |

```bash
# Set SUID on an executable
chmod u+s /usr/bin/some_program

# Set sticky bit on a shared directory
chmod +t /tmp/shared/

# Combined: SUID + rwxr-xr-x
chmod 4755 program

# Combined: sticky + rwxrwxrwx
chmod 1777 /tmp
```

---

## Quick Reference

### Symbolic Mode Quick Reference

```bash
chmod u+x file        # Add execute for owner
chmod g-w file        # Remove write for group
chmod o=r file        # Set others to read-only
chmod a+r file        # Add read for all
chmod u=rwx,g=rx,o=   # Set exact permissions
chmod g=u file        # Copy owner's permissions to group
```

### Numeric Mode Quick Reference

```bash
chmod 777 file        # rwxrwxrwx (dangerous!)
chmod 755 file        # rwxr-xr-x (standard executable)
chmod 644 file        # rw-r--r-- (standard file)
chmod 700 file        # rwx------ (private)
chmod 600 file        # rw------- (very private)
chmod 400 file        # r-------- (read-only)
```

### Viewing Permissions

```bash
ls -l file            # Standard view
ls -la                # All files including hidden
stat -c "%a %n" file  # Show numeric permissions
stat file             # Full file statistics
```

---

## Hands-On Lab Exercises

### 🧪 Exercise 1: Basic Permission Reading

```bash
# 1. Create a test directory
mkdir ~/chmod_lab
cd ~/chmod_lab

# 2. Create some test files
touch file1.txt file2.sh file3.data

# 3. List them with permissions
ls -l

# 4. Answer these questions:
#    - What are the default permissions?
#    - Who is the owner?
#    - Who is the group?
```

### 🧪 Exercise 2: Symbolic Mode Practice

```bash
cd ~/chmod_lab

# 1. Add execute permission for owner on file2.sh
chmod u+x file2.sh
ls -l file2.sh

# 2. Remove read permission for others on file1.txt
chmod o-r file1.txt
ls -l file1.txt

# 3. Set exact permissions: owner=rwx, group=rx, others=r
chmod u=rwx,g=rx,o=r file3.data
ls -l file3.data

# 4. Add write permission for group on all files
chmod g+w *
ls -l
```

### 🧪 Exercise 3: Numeric Mode Practice

```bash
cd ~/chmod_lab

# 1. Set file1.txt to 644 (rw-r--r--)
chmod 644 file1.txt
ls -l file1.txt

# 2. Set file2.sh to 755 (rwxr-xr-x)
chmod 755 file2.sh
ls -l file2.sh

# 3. Set file3.data to 700 (rwx------)
chmod 700 file3.data
ls -l file3.data

# 4. Set all files to 600 (rw-------)
chmod 600 *
ls -l
```

### 🧪 Exercise 4: Directory Permissions

```bash
cd ~/chmod_lab

# 1. Create a directory
mkdir secrets

# 2. Create a file inside it
echo "top secret" > secrets/note.txt

# 3. Remove execute permission for others
chmod o-x secrets

# 4. Try to cd into it as another user (or check with ls -ld)
ls -ld secrets

# 5. Restore and observe the difference
chmod 755 secrets
ls -ld secrets
```

### 🧪 Exercise 5: Recursive Permissions

```bash
cd ~/chmod_lab

# 1. Create a nested structure
mkdir -p project/src project/docs project/bin
touch project/src/main.py project/docs/readme.md project/bin/run.sh

# 2. Apply 755 to all directories and 644 to all files
find project -type d -exec chmod 755 {} \;
find project -type f -exec chmod 644 {} \;

# 3. Verify
find project -exec ls -ld {} \;
```

### 🧪 Exercise 6: Real-World Scenario

```bash
# Scenario: You've created a web application
mkdir ~/webapp
cd ~/webapp

# Create files
echo "<html>Hello</html>" > index.html
echo "console.log('hi')" > app.js
echo "body { color: black; }" > style.css
mkdir uploads

# Task: Set appropriate permissions
# - HTML/JS/CSS files: readable by everyone, writable by owner
# - uploads directory: owner full control, group can write, others can read
# - All other directories: standard web directory permissions

chmod 644 index.html app.js style.css
chmod 775 uploads
chmod 755 .

# Verify everything
ls -la
ls -ld uploads
```

---

## ✅ Summary Checklist

- [ ] I understand the three permission types: `r`, `w`, `x`
- [ ] I understand the three user classes: `u`, `g`, `o`
- [ ] I can read `ls -l` output and interpret permissions
- [ ] I can use symbolic mode (`chmod u+x`, `chmod go-w`, etc.)
- [ ] I can use numeric mode (`chmod 755`, `chmod 644`, etc.)
- [ ] I know common permission values and when to use them
- [ ] I can apply permissions recursively with `-R`
- [ ] I understand the difference between file and directory permissions
- [ ] I know how to secure sensitive files (SSH keys, passwords)

---

## 📚 Additional Resources

- [Linux File Permissions Explained - Red Hat](https://www.redhat.com/en/blog/linux-file-permissions-explained)
- [chmod Command in Linux - Linuxize](https://linuxize.com/post/chmod-command-in-linux/)
- [Modify File Permissions with chmod - Linode Docs](https://www.akamai.com/docs/guides/modify-file-permissions-with-chmod/)
- [Original Video: Linux for Programmers #4](https://youtu.be/RQLNDekr2wg)

---

*Happy learning! 🐧*
