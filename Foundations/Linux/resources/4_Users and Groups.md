# 4. Users and Groups: Managing Access

## Introduction

Linux is a **multi-user system** — even if you're the only one using it, processes and services often run under different users. As a DevOps engineer, you'll frequently:

- Create **system users** for apps and services (e.g., `nginx`, `postgres`)
- Manage user access to files and commands
- Control **who can do what**

> Understanding users and groups is the first step toward **secure, multi-user systems**.

---

## Users

Every user in Linux has:

- A **username**
- A **home directory** (`/home/username`)
- A **UID** (User ID)
- A **default shell**

**To see who you are:**

```bash
whoami
```

**List all users:**

```bash
cat /etc/passwd
```

**Create a new user:**

```bash
sudo adduser john
```

**Switch to that user:**

```bash
su - john
```

---

## Groups

Groups help you **assign permissions to multiple users at once**.

**View current groups:**

```bash
groups
```

**Add a user to a group:**

```bash
sudo usermod -aG docker john
```

Here, John can now use Docker **without `sudo`**.

### Common Groups

| Group | Purpose |
|-------|---------|
| `sudo` | Can run commands as root |
| `docker` | Can run Docker |
| `www-data` | Used by web servers |

---

## Real-World Example

You have a shared project directory:

```bash
sudo mkdir /var/shared_project
sudo groupadd devs
sudo chgrp devs /var/shared_project
sudo chmod 770 /var/shared_project
```

Now add your team members to the `devs` group:

```bash
sudo usermod -aG devs alice
sudo usermod -aG devs bob
```

Everyone in `devs` can now read/write in `/var/shared_project`.

---

## The Superuser

The **root user** has full control. You should **avoid using root directly**, and instead use:

```bash
sudo somecommand
```

This runs a command with admin privileges — safely and with an audit trail.
