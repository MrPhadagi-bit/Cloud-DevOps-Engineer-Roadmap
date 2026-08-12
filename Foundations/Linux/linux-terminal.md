# 🐧 Linux for Hackers — EP 1: The Terminal

> **A focused reference guide on the Linux Terminal, based on [NetworkChuck's *Linux for Hackers* EP 1](https://youtu.be/VbEx7B_PTOE).**  
> *Everything you need to understand the shell and get hands-on in a real Linux lab.*

---

##  What Is the Terminal?

The **terminal** is your text-based command-line interface (CLI) to the Linux operating system. It is the single most powerful tool in a hacker's arsenal.

Unlike a graphical user interface (GUI) where you click buttons and windows, the terminal lets you talk directly to the computer using typed commands. Every action you can perform in a GUI — and thousands more you cannot — can be done faster and more precisely in the terminal.

> 💡 **Why hackers love the terminal:**
> - **Speed:** Typing a command is faster than navigating menus.
> - **Precision:** You control exactly what happens, with no hidden background processes.
> - **Automation:** You can chain commands, write scripts, and automate entire workflows.
> - **Remote Access:** SSH into servers and hacking labs anywhere in the world.
> - **Tooling:** Nearly every hacking tool (Nmap, Metasploit, Burp Suite) is designed for the terminal.

Think of the terminal as the steering wheel of a race car. The GUI is like a self-driving car — convenient, but you have no real control. The terminal puts you in the driver's seat.

---

## 🔄 Terminal vs. Shell

People often use "terminal" and "shell" interchangeably, but they are two different things:

| Component | What It Is | Analogy |
|-----------|-----------|---------|
| **Terminal** | The graphical window or physical device that displays text and accepts your keystrokes. | The car's dashboard |
| **Shell** | The actual program that interprets the commands you type and translates them into instructions for the computer. | The car's engine |

When you open a terminal window and type `ls`, the **terminal** captures your keystrokes and passes them to the **shell**. The shell reads `ls`, understands you want to list files, executes that instruction, and sends the results back to the terminal to display on your screen.

You can think of it like this:

```
You (type "ls")  →  Terminal (displays text)  →  Shell (interprets & executes)  →  Kernel (does the work)
```

> 🔑 **Key takeaway:** The terminal is the window. The shell is the brain inside it.

---

## 🐚 Common Shells

A **shell** is a command-line interpreter. Linux supports multiple shells, each with its own features and syntax. Here are the three most common ones you will encounter:

---

### Bash (Bourne Again Shell)

| Attribute | Details |
|-----------|---------|
| **Name** | Bourne Again Shell |
| **Default on** | Most Linux distributions (Ubuntu, Debian, Kali Linux, CentOS) |
| **File** | `/bin/bash` |
| **Strengths** | Universal, well-documented, massive community, scripting powerhouse |

Bash is the default shell on the vast majority of Linux systems. If you are just starting out, Bash is where you should begin. Every tutorial, book, and course assumes Bash unless stated otherwise.

Bash supports:
- **Tab completion:** Press `Tab` to auto-complete file names and commands.
- **Command history:** Press `↑` and `↓` to cycle through previously run commands.
- **Scripting:** Write text files (`.sh`) containing multiple commands that can be executed together.
- **Variables & aliases:** Create shortcuts and store data for reuse.

```bash
# Check which shell you are currently using
echo $SHELL

# Output: /bin/bash
```

---

### Zsh (Z Shell)

| Attribute | Details |
|-----------|---------|
| **Name** | Z Shell |
| **Default on** | macOS (since Catalina), Kali Linux (recent versions), many power-user setups |
| **File** | `/bin/zsh` |
| **Strengths** | Highly customizable, better auto-completion, plugin ecosystem (Oh My Zsh) |

Zsh is Bash-compatible but adds powerful features on top. It is the shell of choice for many advanced users and hackers who want a polished, efficient workflow.

What makes Zsh special:
- **Smarter tab completion:** Completes commands, flags, and paths more intelligently.
- **Spelling correction:** Catches typos and suggests fixes.
- **Themes & plugins:** Frameworks like [Oh My Zsh](https://ohmyz.sh/) let you customize prompts, add git integration, syntax highlighting, and more.
- **Shared history:** Commands from all open terminal windows are stored in one history file.

```bash
# Check if Zsh is installed
which zsh

# Switch to Zsh (temporarily)
zsh

# Make Zsh your default shell
chsh -s $(which zsh)
```

> ⚡ **Pro Tip:** Kali Linux now ships with Zsh by default. If you see a fancy prompt with git branch info and colors, you are probably running Zsh.

---

### Fish (Friendly Interactive Shell)

| Attribute | Details |
|-----------|---------|
| **Name** | Friendly Interactive Shell |
| **Default on** | Not default on most distros, but easy to install |
| **File** | `/usr/bin/fish` |
| **Strengths** | Beginner-friendly, autosuggestions, syntax highlighting out of the box |

Fish is designed to be user-friendly right from the start. Unlike Bash and Zsh, it does things automatically that would require plugins or configuration in other shells.

Fish features:
- **Autosuggestions:** As you type, Fish suggests completions in gray text based on your history. Press `→` to accept.
- **Syntax highlighting:** Commands turn green when valid, red when invalid, as you type.
- **Web-based configuration:** Run `fish_config` to open a browser GUI for customizing colors and prompts.
- **Easy-to-read documentation:** Built-in `help` command with clean, web-style pages.

```bash
# Install Fish
sudo apt install fish    # Debian/Ubuntu/Kali
sudo dnf install fish    # Fedora

# Start Fish
fish

# Open the web configurator
fish_config
```

> 🐟 **Note:** Fish is not fully POSIX-compatible with Bash. Some Bash scripts may need minor tweaks to run in Fish. It is excellent for interactive use, but many hackers still write scripts in Bash for portability.

---

### Shell Comparison Summary

| Feature | Bash | Zsh | Fish |
|---------|------|-----|------|
| Default on most Linux | ✅ | ⚠️ (some) | ❌ |
| Default on macOS | ❌ (older) | ✅ (Catalina+) | ❌ |
| Tab completion | Basic | Advanced | Advanced |
| Syntax highlighting | ❌ (needs plugin) | ❌ (needs plugin) | ✅ Built-in |
| Autosuggestions | ❌ (needs plugin) | ❌ (needs plugin) | ✅ Built-in |
| Scripting portability | ✅ Best | ✅ Good | ⚠️ Moderate |
| Beginner-friendly | ⚠️ Moderate | ⚠️ Moderate | ✅ Best |
| Customization | Moderate | Excellent (Oh My Zsh) | Good (web UI) |

> 🎯 **Recommendation for beginners:** Start with **Bash** to build a solid foundation. Once comfortable, experiment with **Zsh** for power-user features or **Fish** for a smoother learning curve.

---

## 🚪 How to Access a Terminal

### On Kali Linux / Parrot OS
- Click the terminal icon in the taskbar, or
- Press `Ctrl + Alt + T`

### On Ubuntu / Debian
- Press `Ctrl + Alt + T`, or
- Search "Terminal" in the applications menu

### On macOS
- Open **Terminal.app** from `Applications > Utilities`, or
- Press `Cmd + Space`, type "Terminal", and hit `Enter`

### On Windows
- **WSL (Windows Subsystem for Linux):** Open your installed Linux distro from the Start Menu
- **Git Bash:** Install Git for Windows and use the included Bash terminal
- **PowerShell / CMD:** Not a true Linux shell, but WSL is the recommended route

### Remote Access via SSH
Once you have a Linux machine running (locally or in the cloud), you can connect remotely:

```bash
ssh username@ip-address
```

You will be prompted for a password, and then you are dropped straight into a shell on that remote machine — no GUI needed.

---

## 📝 The Prompt Explained

When you open a terminal, you see a line of text before your cursor. That is the **prompt**, and it tells you important context about where you are and who you are.

A typical Bash prompt looks like this:

```
kali@kali-vm:~$
```

Let's break it down:

| Part | Meaning |
|------|---------|
| `kali` | The **username** you are logged in as |
| `@` | Separator |
| `kali-vm` | The **hostname** (name of the machine) |
| `:` | Separator |
| `~` | Your current **directory** (`~` is shorthand for your home folder) |
| `$` | Indicates you are a **regular user** |
| `#` | (If you see this) You are logged in as **root** (the superuser) |

Examples:

```
kali@kali-vm:~$          # Regular user, in home directory
root@kali-vm:/etc#       # Root user, in /etc directory
htb-student@htb-academy-lab:~$   # HTB lab user
```

> ⚠️ **The `#` symbol is a warning.** When you are root, you have unlimited power. One wrong command can destroy the entire system. Always double-check before pressing Enter as root.

---

## 🎮 Your First Commands

Before jumping into the lab, try these commands in any terminal to get comfortable:

```bash
# Who am I?
whoami

# What is this machine called?
hostname

# Where am I in the filesystem?
pwd

# What files are here?
ls

# What is my shell?
echo $SHELL

# Clear the screen
clear

# View command history
history
```

> 💡 **Pro Tip:** Press `Tab` to auto-complete commands and file paths. Press `↑` to recall the last command. Press `Ctrl + C` to cancel a running command. Press `Ctrl + L` to clear the screen.

---

## 🧪 Step-by-Step: Log Into the FREE HTB Academy Linux Lab

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

## 🎯 Summary

- The **terminal** is the window; the **shell** is the interpreter.
- **Bash** is the standard — learn it first.
- **Zsh** adds power-user features and customization.
- **Fish** is the most beginner-friendly with built-in autosuggestions.
- The **prompt** tells you who you are, what machine you are on, and where you are in the filesystem.
- The **HTB Academy Linux Fundamentals** module is a free, hands-on lab. Use the Pwnbox or SSH into lab machines to practice real commands.

> ☕ *"Coffee + Linux = Hacker Fuel"* — NetworkChuck

**Based on:** NetworkChuck's *Linux for Hackers* EP 1  
**Lab Link:** [https://ntck.co/htbacad](https://ntck.co/htbacad)
