# 🔐 `chmod` and File Permissions — Complete Guide

> A comprehensive, beginner-friendly reference for understanding and managing Linux/Unix file permissions using `chmod`.
>
> 📺 *Based on the tutorial: [chmod & File Permissions Explained](https://youtu.be/RQLNDekr2wg?si=PeAhNbF_MV_mRvel)*

---

## 📑 Table of Contents

1. [What Are File Permissions?](#1-what-are-file-permissions)
2. [The Three Permission Types](#2-the-three-permission-types-r-w-x)
3. [The Three User Classes](#3-the-three-user-classes-owner-group-others)
4. [Reading `ls -l` Output](#4-reading-ls--l-output)
5. [Octal (Numeric) Notation](#5-octal-numeric-notation)
6. [Symbolic Notation](#6-symbolic-notation)
7. [Special Permissions (SetUID, SetGID, Sticky Bit)](#7-special-permissions-setuid-setgid-sticky-bit)
8. [Practical Examples](#8-practical-examples)
9. [Common Permission Values](#9-common-permission-values)
10. [Recursive Changes](#10-recursive-changes)
11. [Quick Reference Tables](#11-quick-reference-tables)
12. [FAQ](#12-faq)

---

## 1. What Are File Permissions?

Every file and directory on a Linux/Unix system has an **owner**, a **group**, and a set of **permissions** that control who can:

- **Read** the contents
- **Write** (modify) the contents
- **Execute** (run) the file

These permissions are the foundation of Linux security. They prevent unauthorized users from accessing, modifying, or running files they shouldn't.

> 💡 **Key Idea:** Permissions are divided into three groups (called *classes*): the **owner** of the file, the **group** that owns the file, and **everyone else** on the system.

---

## 2. The Three Permission Types: `r`, `w`, `x`

| Symbol | Name | For Files | For Directories |
|--------|------|-----------|-----------------|
| **r** | **Read** | View the file's contents | List files inside the directory (`ls`) |
| **w** | **Write** | Modify or delete the file | Create, rename, or delete files inside |
| **x** | **Execute** | Run the file as a program | Enter the directory (`cd`) |

### 🔍 Important Notes:

- **Execute on directories** does NOT mean you can list files — it means you can *enter* the directory. To list files, you also need **read** (`r`).
- **Write on directories** means you can create, rename, or delete files *inside* that directory — even if you don't own those files!
- For a script or program to run, it needs both **read** (`r`) and **execute** (`x`) permissions.

---

## 3. The Three User Classes: Owner, Group, Others

| Class | Symbol | Description |
|-------|--------|-------------|
| **User (Owner)** | `u` | The user account that owns the file |
| **Group** | `g` | Members of the file's assigned group |
| **Others** | `o` | Everyone else on the system (not owner, not in group) |
| **All** | `a` | Shortcut for all three classes (`u`, `g`, and `o`) |

> ⚠️ **Common Mistake:** `o` stands for **others** (everyone else), NOT **owner**. The owner is `u`.

---

## 4. Reading `ls -l` Output

When you run `ls -l` (or `ls -la` to include hidden files), you see something like this:

```bash
$ ls -l
-rwxr-xr--  1 alice developers  1234 Aug 29 10:00 myscript.sh
drwxr-xr-x  2 alice developers  4096 Aug 29 09:30 myfolder
lrwxrwxrwx  1 alice developers    12 Aug 29 08:00 mylink -> /path/to/file
```

### 🔍 Breaking Down the First 10 Characters

```
-rwxr-xr--
│└┬┘└┬┘└┬┘
│ │  │  │
│ │  │  └── Others permissions: r--
│ │  └───── Group permissions:  r-x
│ └──────── Owner permissions:   rwx
└────────── File type: - (regular file)
```

| Position | Meaning |
|----------|---------|
| **1st char** | File type: `-` = regular file, `d` = directory, `l` = symbolic link, `c` = character device, `b` = block device |
| **2–4** | Owner (`u`) permissions: `r`, `w`, `x` |
| **5–7** | Group (`g`) permissions: `r`, `w`, `x` |
| **8–10** | Others (`o`) permissions: `r`, `w`, `x` |

### Example Breakdown: `-rwxr-xr--`

| Class | Permissions | Meaning |
|-------|-------------|---------|
| Owner (`u`) | `rwx` | Can read, write, and execute |
| Group (`g`) | `r-x` | Can read and execute, but NOT write |
| Others (`o`) | `r--` | Can only read |

---

## 5. Octal (Numeric) Notation

Octal notation uses **three digits** (0–7) to represent permissions. Each digit corresponds to one user class (owner, group, others).

### Permission Values

| Value | Binary | Permission | Description |
|-------|--------|------------|-------------|
| **0** | `000` | `---` | No permissions |
| **1** | `001` | `--x` | Execute only |
| **2** | `010` | `-w-` | Write only |
| **3** | `011` | `-wx` | Write + Execute |
| **4** | `100` | `r--` | Read only |
| **5** | `101` | `r-x` | Read + Execute |
| **6** | `110` | `rw-` | Read + Write |
| **7** | `111` | `rwx` | Read + Write + Execute (Full) |

### How to Calculate

Each permission has a numeric value:

| Permission | Value |
|------------|-------|
| Read (`r`) | **4** |
| Write (`w`) | **2** |
| Execute (`x`) | **1** |

Add them up for each class:

```
Owner:  rwx = 4 + 2 + 1 = 7
Group:  r-x = 4 + 0 + 1 = 5
Others: r-- = 4 + 0 + 0 = 4

Result: chmod 754 filename
```

### Syntax

```bash
chmod <octal-value> <filename>
```

### Examples

```bash
chmod 755 myscript.sh    # rwxr-xr-x  (owner full, group/others read+execute)
chmod 644 myfile.txt     # rw-r--r--  (owner read+write, group/others read only)
chmod 700 secret.key     # rwx------  (owner only, full access)
chmod 777 shared.txt     # rwxrwxrwx  (everyone full access — use with caution!)
chmod 600 private.txt    # rw-------  (owner read+write only)
```

---

## 6. Symbolic Notation

Symbolic notation uses letters and operators to **modify** existing permissions rather than replacing them entirely.

### Syntax

```bash
chmod [classes][operator][permissions] <filename>
```

### Classes

| Symbol | Class |
|--------|-------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (`ugo`) |

### Operators

| Symbol | Meaning |
|--------|---------|
| `+` | **Add** the permission |
| `-` | **Remove** the permission |
| `=` | **Set exactly** these permissions (removes others) |

### Permissions

| Symbol | Meaning |
|--------|---------|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `X` | Execute only if file is a directory OR already has execute for some user |
| `s` | SetUID or SetGID |
| `t` | Sticky bit |

### Examples

```bash
chmod u+x myscript.sh        # Add execute for owner
chmod g-w myfile.txt         # Remove write for group
chmod o+r myfile.txt         # Add read for others
chmod a+x myprogram          # Add execute for everyone (same as +x)
chmod u=rwx,g=rx,o=r myfile  # Set exact permissions
chmod go= myfile.txt         # Remove ALL permissions for group and others
chmod g=u myfile.txt         # Give group same permissions as owner
```

### Combining Multiple Changes

Use commas to separate multiple changes:

```bash
chmod u+x,go-w,o+r myfile.txt
```

---

## 7. Special Permissions (SetUID, SetGID, Sticky Bit)

Beyond the basic `rwx`, there are three **special permissions** that affect how files and directories behave.

### 7.1 SetUID (`s` on owner execute) — Numeric: `4000`

When set on an **executable file**, the file runs with the **owner's privileges**, not the user who launched it.

```bash
chmod u+s myprogram      # Symbolic
chmod 4755 myprogram     # Octal (4 = SetUID)
```

> 🔒 **Security Note:** SetUID is powerful and potentially dangerous. It's used by commands like `passwd` (which needs root access to modify `/etc/shadow`).

**In `ls -l`:**
```
-rwsr-xr-x  1 root root  ...  myprogram   # 's' replaces 'x' in owner position
-rwSr-xr-x  1 root root  ...  myprogram   # 'S' = SetUID set but no execute
```

### 7.2 SetGID (`s` on group execute) — Numeric: `2000`

- **On executable files:** Runs with the **group's privileges**.
- **On directories:** New files created inside inherit the **directory's group** (not the user's primary group). Useful for shared project directories.

```bash
chmod g+s mydir          # Symbolic
chmod 2775 mydir         # Octal (2 = SetGID)
```

**In `ls -l`:**
```
-rwxr-sr-x  1 alice devteam  ...  myprogram   # 's' replaces 'x' in group position
```

### 7.3 Sticky Bit (`t` on others execute) — Numeric: `1000`

When set on a **directory**, users can only delete or rename **their own files**, even if they have write access to the directory.

> Classic example: `/tmp` directory — everyone can write, but you can't delete someone else's files.

```bash
chmod +t mydir           # Symbolic
chmod 1777 mydir         # Octal (1 = Sticky Bit)
```

**In `ls -l`:**
```
drwxrwxrwt  10 root root  ...  /tmp   # 't' replaces 'x' in others position
```

### Four-Digit Octal

When using special permissions, prepend a fourth digit:

```bash
chmod 4755 file    # SetUID + rwxr-xr-x
chmod 2775 dir     # SetGID + rwxrwxr-x
chmod 1777 dir     # Sticky Bit + rwxrwxrwx
chmod 6755 file    # SetUID + SetGID + rwxr-xr-x
```

| Special Bit | Numeric Prefix | Symbol |
|-------------|----------------|--------|
| SetUID | `4` | `u+s` |
| SetGID | `2` | `g+s` |
| Sticky Bit | `1` | `+t` |

---

## 8. Practical Examples

### Make a Script Executable

```bash
chmod +x myscript.sh
# or
chmod 755 myscript.sh
```

### Secure a Private Key or Password File

```bash
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.mysecrets
```

### Share a File Read-Only with Everyone

```bash
chmod 644 report.pdf
```

### Create a Shared Directory (SetGID)

```bash
mkdir /shared/project
chgrp devteam /shared/project
chmod 2775 /shared/project
# Now all new files inherit the 'devteam' group
```

### Make a Directory World-Writable but Safe (Sticky Bit)

```bash
chmod 1777 /shared/dropbox
# Everyone can write, but only delete their own files
```

### Remove All Permissions for Others

```bash
chmod o-rwx myfile.txt
# or
chmod 770 myfile.txt
```

### Copy Owner Permissions to Group

```bash
chmod g=u myfile.txt
```

---

## 9. Common Permission Values

| Value | Notation | Use Case |
|-------|----------|----------|
| `777` | `rwxrwxrwx` | ⚠️ Everyone full access. Rarely appropriate. |
| `755` | `rwxr-xr-x` | Standard for executables and directories. |
| `750` | `rwxr-x---` | Owner full, group read+execute, others nothing. |
| `700` | `rwx------` | Private executables (e.g., personal scripts). |
| `644` | `rw-r--r--` | Standard for regular files (readable by all, writable by owner). |
| `640` | `rw-r-----` | Readable by owner and group only. |
| `600` | `rw-------` | Private files (SSH keys, passwords). |
| `666` | `rw-rw-rw-` | Shared writable files (use with caution). |
| `777` + sticky | `1777` | Shared temp directories like `/tmp`. |

---

## 10. Recursive Changes

Use the `-R` flag to apply permissions to a directory and **everything inside it**:

```bash
chmod -R 755 myproject/
chmod -R u+rwX,go-rwx myproject/   # Capital X: only dirs and already-executable files
```

> ⚠️ **Warning:** Be extremely careful with `chmod -R` on system directories like `/`, `/etc`, or `/usr`. You can break your system!

### Safer Recursive: Use `find`

Apply different permissions to files vs. directories:

```bash
# Make all directories accessible
find myproject -type d -exec chmod 755 {} +

# Make all files readable (but not executable)
find myproject -type f -exec chmod 644 {} +
```

---

## 11. Quick Reference Tables

### Octal to Symbolic Conversion

| Octal | Owner | Group | Others | Symbolic Equivalent |
|-------|-------|-------|--------|---------------------|
| `000` | `---` | `---` | `---` | `a=` |
| `644` | `rw-` | `r--` | `r--` | `u=rw,go=r` |
| `755` | `rwx` | `r-x` | `r-x` | `u=rwx,go=rx` |
| `700` | `rwx` | `---` | `---` | `u=rwx,go=` |
| `777` | `rwx` | `rwx` | `rwx` | `a=rwx` |
| `750` | `rwx` | `r-x` | `---` | `u=rwx,g=rx,o=` |
| `600` | `rw-` | `---` | `---` | `u=rw,go=` |

### Symbolic Notation Quick Reference

| Command | Effect |
|---------|--------|
| `chmod +x file` | Make file executable for all |
| `chmod u+x file` | Make file executable for owner |
| `chmod go-w file` | Remove write for group and others |
| `chmod a=r file` | Set read-only for everyone |
| `chmod 644 file` | Standard file permissions |
| `chmod 755 file` | Standard executable/directory permissions |
| `chmod -R 755 dir` | Recursively apply to directory |

---

## 12. FAQ

### Q: What's the difference between `chmod +x` and `chmod a+x`?

**A:** They are the same. If no class is specified, `a` (all) is assumed.

### Q: Why is `chmod 777` dangerous?

**A:** It grants full read, write, and execute permissions to **everyone** on the system. Any user (including compromised accounts) can modify or delete your files. Use the **principle of least privilege** — give only the permissions that are actually needed.

### Q: What's the difference between `chmod` and `chown`?

**A:**
- `chmod` = **CH**ange **MOD**e — changes **permissions** (rwx).
- `chown` = **CH**ange **OWN**er — changes **who owns** the file.
- `chgrp` = **CH**ange **GR**ou**P** — changes the file's group.

### Q: What does `chmod g=u` do?

**A:** It sets the group's permissions to **exactly match** the owner's current permissions.

### Q: What is `umask` and how does it relate to `chmod`?

**A:** `umask` sets the **default permissions** for newly created files. It's subtractive — a `umask` of `022` means new files get `644` (666 - 022) and directories get `755` (777 - 022). `chmod` changes permissions on **existing** files.

### Q: Can I use `chmod` on symbolic links?

**A:** `chmod` changes the permissions of the **target file**, not the link itself. Symbolic links typically show `lrwxrwxrwx` — these are "dummy" permissions.

### Q: What is the `X` (capital X) permission?

**A:** It's a conditional execute. When used with `+` (e.g., `chmod -R a+rX .`), it adds execute permission to **directories** and **files that already have execute permission for some user**. It's safer than `+x` for recursive operations because it won't make regular files executable.

---

## 🎯 Summary

| Concept | Key Takeaway |
|---------|-------------|
| **Permissions** | `r` (read=4), `w` (write=2), `x` (execute=1) |
| **Classes** | `u` (owner), `g` (group), `o` (others), `a` (all) |
| **Octal** | Three digits: owner-group-others. Add 4+2+1 for each. |
| **Symbolic** | Use `+`, `-`, `=` with classes and permissions. |
| **Special** | SetUID (`4000`), SetGID (`2000`), Sticky Bit (`1000`) |
| **Recursive** | Use `-R` flag, or `find` for more control. |
| **Security** | Use `600` for private files, `755` for executables, avoid `777`. |

---

## 📚 Additional Resources

- [Linux File Permissions Explained — Red Hat](https://www.redhat.com/en/blog/linux-file-permissions-explained)
- [chmod Manual — Wikipedia](https://en.wikipedia.org/wiki/Chmod)
- [chmod Guide — Akamai/Linode](https://www.akamai.com/docs/guides/modify-file-permissions-with-chmod/)
- [Unix File Permissions — NERSC](https://docs.nersc.gov/filesystems/unix-file-permissions/)
- 📺 [Video Tutorial Reference](https://youtu.be/RQLNDekr2wg?si=PeAhNbF_MV_mRvel)

---

> 💡 **Pro Tip:** Always verify your changes with `ls -l` after running `chmod`. When in doubt, start restrictive and loosen permissions as needed. Security is easier to maintain when you begin with the principle of least privilege!

---

*Happy learning! 🐧*
