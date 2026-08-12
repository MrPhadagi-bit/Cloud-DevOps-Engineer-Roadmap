##  Step-by-Step: Log Into the FREE HTB Academy Linux Lab

NetworkChuck recommends the **Hack The Box (HTB) Academy** as a free, hands-on way to practice Linux. The Linux Fundamentals module is a **Tier 0 course**, meaning it is completely free to access.

> 🔗 **Direct Link:** [https://ntck.co/htbacad](https://ntck.co/htbacad)

### What You Get for Free

| Feature | Free Tier Details |
|---------|-------------------|
| **Linux Fundamentals Module** | Full access to the Tier 0 course |
| **In-Browser Pwnbox** | A Kali Linux VM that runs inside your browser |
| **Lab Machines** | Dedicated practice machines you can SSH into |
| **Daily Limit** | 1 Pwnbox spawn per day (60-minute sessions) |
| **Cost** | $0 |

---

### Step 1: Create Your HTB Academy Account

1. Go to [https://academy.hackthebox.com](https://academy.hackthebox.com) (or use NetworkChuck's link: [https://ntck.co/htbacad](https://ntck.co/htbacad)).
2. Click **"Start for Free"** or **"Sign Up"**.
3. Fill in your details:
   - Username
   - Email address
   - Password
4. Verify your email address via the confirmation link sent to your inbox.
5. Log in to the HTB Academy dashboard.

---

### Step 2: Enroll in the Linux Fundamentals Module

1. From the dashboard, use the search bar or browse the module catalog.
2. Find **"Linux Fundamentals"** (it is a Tier 0 module).
3. Click **"Start Module"** or **"Enroll"** — no payment required.
4. You will see a list of sections:
   - Linux Structure
   - Linux Distributions
   - Introduction to Shell
   - Prompt Description
   - Getting Help
   - System Information
   - Navigation
   - Working with Files and Directories
   - ...and more

---

### Step 3: Launch the In-Browser Pwnbox

The **Pwnbox** is a pre-configured Kali Linux virtual machine that runs entirely in your browser. This is the easiest way to practice without installing anything on your computer.

1. Inside the Linux Fundamentals module, look for the **"Start Pwnbox"** or **"Spawn Instance"** button.
2. Click it and wait 30–60 seconds for the VM to boot.
3. A browser-based terminal window will appear — you are now inside a real Kali Linux shell.
4. The Pwnbox session lasts **60 minutes** on the free tier. Use your time wisely.

> 🖥️ **What is Pwnbox?** It is a fully functional Kali Linux environment with all hacking tools pre-installed. You have a terminal, a file manager, and a web browser inside it. It is perfect for following along with the Linux Fundamentals lessons.

---

### Step 4: Connect to the Lab Machine via SSH (Optional)

Some lessons in the module require you to connect to a dedicated lab machine (not the Pwnbox itself). You will be given an IP address and credentials.

**Typical lab credentials:**

| Field | Value |
|-------|-------|
| **Username** | `htb-student` |
| **Password** | `HTB_@cademy_stdnt!` |
| **IP Address** | Provided in the module (e.g., `10.129.x.x`) |

**Connection steps:**

1. Make sure your Pwnbox is running (or connect via HTB VPN from your local machine).
2. Open a terminal.
3. Run the SSH command:

```bash
ssh htb-student@<LAB-IP-ADDRESS>
```

4. When prompted, type the password: `HTB_@cademy_stdnt!`
   - **Note:** Passwords do not show on screen as you type — this is normal for security.
5. Press `Enter`. You should now see a prompt like:

```
htb-student@htb-academy-lab:~$
```

6. You are now logged into the lab machine. Follow the module instructions and practice the commands.

> 🔐 **VPN Alternative:** If you prefer to use your own local Kali VM instead of the browser Pwnbox, you can download an OpenVPN connection file from your HTB Academy profile. Connect with `sudo openvpn your-file.ovpn`, then SSH into the lab machine from your local terminal.

---

### Step 5: Practice & Follow the Lessons

With your terminal open (either in Pwnbox or via SSH), work through each section of the Linux Fundamentals module:

1. **Read the lesson text** on the left side of the HTB Academy page.
2. **Run the commands** in your terminal on the right (or in your own terminal).
3. **Answer the knowledge checks** at the end of each section to reinforce learning.
4. **Take notes** on commands that are new to you.

> ⏰ **Free Tier Tip:** You get one Pwnbox session per day (60 minutes). Plan your study time. If you run out of time, you can still read the lessons and take notes — just spawn a new Pwnbox the next day to practice.

---

### Troubleshooting

| Problem | Solution |
|---------|----------|
| "Connection timed out" when SSHing | Make sure the Pwnbox is running. The lab machine may only be reachable from within the HTB network (Pwnbox or VPN). |
| Password not working | Double-check for typos. The password is case-sensitive: `HTB_@cademy_stdnt!` |
| Pwnbox won't start | Clear your browser cache, disable ad blockers, or try a different browser (Chrome/Firefox recommended). |
| "Initialization Sequence Completed" but can't ping | Wait 30 seconds after the VPN connects, then try again. |

---

## 📋 Quick Reference

| Task | Command |
|------|---------|
| Open terminal | `Ctrl + Alt + T` (Linux) |
| Check current user | `whoami` |
| Check hostname | `hostname` |
| Check current shell | `echo $SHELL` |
| Check current directory | `pwd` |
| List files | `ls` |
| Clear screen | `clear` or `Ctrl + L` |
| Cancel command | `Ctrl + C` |
| Auto-complete | `Tab` |
| Recall last command | `↑` |
| SSH into a machine | `ssh username@ip-address` |
| Switch to Bash | `bash` |
| Switch to Zsh | `zsh` |
| Switch to Fish | `fish` |

---

##  Summary

- The **terminal** is the window; the **shell** is the interpreter.
- **Bash** is the standard — learn it first.
- **Zsh** adds power-user features and customization.
- **Fish** is the most beginner-friendly with built-in autosuggestions.
- The **prompt** tells you who you are, what machine you are on, and where you are in the filesystem.
- The **HTB Academy Linux Fundamentals** module is a free, hands-on lab. Use the Pwnbox or SSH into lab machines to practice real commands.
