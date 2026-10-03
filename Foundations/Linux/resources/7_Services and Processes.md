# 7. Services and Processes: Keeping Things Running

## Introduction

When you start a web server, a database, or even a background script — it becomes a **process** running on your system.

If you want these services to:

- **Start automatically** on boot
- **Restart if they crash**
- Be **managed securely** by the system

…then you'll use **services**, managed by a tool called **systemd** (most modern Linux systems use it).

> Mastering services means you can manage what's running on your system — and that's **DevOps gold**.

---

## Processes vs Services

| Term | Description |
|------|-------------|
| **Process** | An individual running task (e.g., your Python script). |
| **Service** | A managed process under `systemd` (e.g., Nginx, Docker). |

**List processes:**

```bash
ps aux          # View all running processes
top             # Real-time view of resource usage
htop            # Enhanced top (install with `sudo apt install htop`)
```

**Kill a process:**

```bash
kill <PID>
kill -9 <PID>   # Forcefully stop (only if needed)
```

**Find a process:**

```bash
ps aux | grep nginx
```

---

## Managing Services with `systemctl`

To interact with services:

| Action | Command |
|--------|---------|
| Start | `sudo systemctl start nginx` |
| Stop | `sudo systemctl stop nginx` |
| Restart | `sudo systemctl restart nginx` |
| Check status | `sudo systemctl status nginx` |
| Enable on boot | `sudo systemctl enable nginx` |
| Disable | `sudo systemctl disable nginx` |

**Example:**

```bash
sudo systemctl status docker
```

Shows if Docker is running, enabled at boot, and logs any failures.

---

## Enable Autostart

Want your service to **auto-start after a reboot**?

```bash
sudo systemctl enable yourservice
```

**Disable it from boot:**

```bash
sudo systemctl disable yourservice
```

> This is crucial for production — you don't want to manually restart everything after a crash.

---

## Real-World Example: Restarting a Failed App

Let's say your web server crashed, and you want to:

1. Check its status
2. Restart it
3. View logs

```bash
sudo systemctl status apache2
sudo systemctl restart apache2
journalctl -u apache2 --since today
```

This helps you troubleshoot and get back online quickly.

---

## Bonus Tip: Kill Unresponsive Services

Sometimes a service hangs. You can stop it hard:

```bash
sudo systemctl stop nginx
sudo killall nginx
```

Or reboot the whole system (last resort):

```bash
sudo reboot
```
