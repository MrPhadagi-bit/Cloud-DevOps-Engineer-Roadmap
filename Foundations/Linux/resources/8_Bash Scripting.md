# 8. Bash Scripting: Automate Everything

## Introduction

Tired of repeating the same commands every time you spin up a server, install dependencies, or deploy an app?

That's where **Bash scripting** comes in. With just a few lines, you can automate tasks like:

- Installing packages
- Backing up files
- Deploying code
- Running cron jobs
- Checking service health

> Bash is your **Linux automation superpower**. DevOps without Bash is like a pilot without a checklist.

---

## What Is a Bash Script?

A Bash script is simply a **file with Linux commands**, executed in order, line by line.

**Sample:**

```bash
#!/bin/bash

echo "Updating system..."
sudo apt update && sudo apt upgrade -y
echo "Done!"
```

- The first line `#!/bin/bash` is called a **shebang** — it tells the system to use Bash to interpret the file.
- `echo` prints text to the screen.

---

## How to Create & Run a Script

1. **Create the file:**

   ```bash
   nano myscript.sh
   ```

2. Add your commands, then save (**Ctrl+O**, **Enter**, **Ctrl+X** in nano).

3. **Make it executable:**

   ```bash
   chmod +x myscript.sh
   ```

4. **Run it:**

   ```bash
   ./myscript.sh
   ```

---

## Add Logic: Conditions and Loops

You can add intelligence to your scripts:

### Conditionals

```bash
if [ -f /etc/passwd ]; then
  echo "User file exists."
else
  echo "File missing!"
fi
```

### Loops

```bash
for file in *.log
do
  echo "Found log file: $file"
done
```

### Variables

```bash
name="Nimesha"
echo "Hello, $name!"
```

---

## Real-World Example: Deployment Script

```bash
#!/bin/bash

echo "Deploying Flask App..."

cd /home/ubuntu/myapp
git pull origin main

sudo systemctl restart flaskapp
echo "Deployment complete!"
```

**This script:**

1. Navigates to the project directory
2. Pulls the latest code from GitHub
3. Restarts the Flask service

> 💡 You could schedule this with a **cron job** (covered in Section 10).

---

## Add Error Handling

```bash
#!/bin/bash
echo "Starting backup..."

tar -czf /backup/home.tar.gz /home || {
  echo "Backup failed!"
  exit 1
}

echo "Backup complete."
```

The `||` ensures you catch errors if something goes wrong.

---

## Pro Tips

- Always start with `#!/bin/bash`
- Use `set -e` to make the script exit on any error
- Comment your code with `#` so future-you can understand it
- Don't run destructive commands (like `rm -rf`) without confirmation
