# 6. Installing Software: Package Managers

## Introduction

Linux doesn't have an "App Store" like Windows or macOS. Instead, it uses **package managers** — powerful tools that let you:

- Install new software
- Remove it cleanly
- Update it safely
- Manage dependencies automatically

Whether you're setting up Docker, Nginx, Python tools, or even VS Code — it all goes through your package manager.

---

## Popular Package Managers

| Package Manager | Used By |
|-----------------|---------|
| **APT** (Advanced Package Tool) | Debian-based systems (Ubuntu, Kali, etc.) |
| **YUM / DNF** | Red Hat-based systems (CentOS, Fedora, RHEL) |
| **Zypper** | SUSE |
| **Pacman** | Arch Linux |

> In DevOps, **Ubuntu is most common** — so we'll use **APT** examples here.

---

## Installing Software

```bash
sudo apt update          # Refreshes package list
sudo apt install nginx   # Installs Nginx web server
```

### Why `apt update`?

This pulls the **latest list of available software and versions** from your configured repositories. Think of it as syncing your app store.

**Want to install Git?**

```bash
sudo apt install git
```

**Need Python tools?**

```bash
sudo apt install python3-pip
```

---

## Updating Software

```bash
sudo apt upgrade
```

Upgrades all installed packages to the latest versions.

**Upgrade everything in one go:**

```bash
sudo apt update && sudo apt upgrade -y
```

The `-y` flag automatically says "yes" to all prompts — useful in scripts.

---

## Removing Software

```bash
sudo apt remove nginx
```

This removes the program, but **keeps config files**.

**To delete everything (including config):**

```bash
sudo apt purge nginx
```

**Clean up unused files:**

```bash
sudo apt autoremove
```

---

## Real-World DevOps Example: Installing Docker

Let's say you want to install Docker:

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

**Verify it's working:**

```bash
docker --version
```

Done! You're now ready to build containers.

---

## Adding External Repositories (Advanced)

Some software (like Node.js or VS Code) isn't in the default list. You can add **third-party repositories** like this:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

This allows you to access more up-to-date or non-default software safely.
