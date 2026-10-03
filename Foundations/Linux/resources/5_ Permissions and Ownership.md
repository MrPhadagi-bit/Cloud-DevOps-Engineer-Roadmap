# 5. Permissions and Ownership: Controlling Access

## Introduction

Linux uses a **permission system** to control who can read, write, or execute a file or directory. Without proper permissions:

- Your app might not start.
- Logs may fail to write.
- Hackers might exploit loose access.

> Understanding permissions is **non-negotiable** for system security and reliability.

---

## Permission Breakdown: `-rwxr-xr--`

This is what you see when you run:

```bash
ls -l somefile.txt
```

**Sample output:**

```
-rwxr-xr-- 1 nimesha devs 1200 Jul 23 12:00 somefile.txt
```

**Breakdown of `-rwxr-xr--`:**

| Position | Value | Meaning |
|----------|-------|---------|
| First character | `-` | Regular file (`d` = directory) |
| Next 3 | `rwx` | Owner can **read, write, execute** |
| Next 3 | `r-x` | Group can **read, execute** |
| Final 3 | `r--` | Others can **only read** |

**So:**

- `nimesha` (owner) has full access
- Group `devs` can read and execute
- Everyone else can only read

---

## Numeric Permissions: 777, 755, 644

Every permission group is a number:

- `r` = 4
- `w` = 2
- `x` = 1

**Add them up:**

| Permission | Calculation | Value |
|------------|-------------|-------|
| `rwx` | 4+2+1 | 7 |
| `rw-` | 4+2 | 6 |
| `r--` | 4 | 4 |

### Common Combinations

| Number | Meaning |
|--------|---------|
| `755` | Owner full (7), group and others can read/execute (5) |
| `644` | Owner can read/write, others can only read |
| `777` | Everyone can do everything ❌ (**avoid in production**) |

---

## Changing Permissions

```bash
chmod 755 script.sh
```

Gives owner full rights, others read/execute only.

**Make a script executable:**

```bash
chmod +x deploy.sh
```

---

## Changing Ownership

To change the file's owner and group:

```bash
sudo chown alice:devs report.txt
```

Use `-R` for folders:

```bash
sudo chown -R www-data:www-data /var/www/html/
```

---

## Real-World Example: Fixing a Broken Web App

You deployed a Flask app and got a **403 Forbidden** error.

**Solution:**

```bash
sudo chown -R www-data:www-data /var/www/myapp
sudo chmod -R 755 /var/www/myapp
```

You're giving ownership to the web server user and proper permissions to read/execute the files.
