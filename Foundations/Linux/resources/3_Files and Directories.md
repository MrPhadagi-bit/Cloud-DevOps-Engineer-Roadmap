# 3. Files and Directories: Creating, Moving, Removing

## Introduction

In Linux, almost everything is a file — configuration files, user data, logs, and even representations of hardware. So, knowing how to **manage files and directories** is the foundation of everything else you'll do.

Whether you're setting up a web server, writing a script, or debugging a container — you'll be:

- Creating and editing files
- Organizing them into folders (directories)
- Moving, renaming, or deleting them

Let's break down these operations, step-by-step.

---

## Create a New Directory (Folder)

```bash
mkdir myproject
```

Creates a new folder called `myproject` in the current location.

**Real-world use:** You're starting a new script-based automation. You create a folder for it:

```bash
mkdir ~/projects/docker-cleanup
cd ~/projects/docker-cleanup
```

You can also create **nested directories**:

```bash
mkdir -p reports/2025/July
```

This creates `reports`, then `2025` inside it, and `July` inside that — all in one command.

---

## Create a New File

```bash
touch notes.txt
```

Creates an empty file named `notes.txt`. Often used to create placeholder files, logs, or quick test files.

**DevOps use case:** Create a placeholder config file before writing:

```bash
touch docker-compose.yml
```

---

## Edit a File

Use a terminal editor like `nano` or `vim`:

```bash
nano notes.txt
```

1. Type your content.
2. Press **Ctrl + O** to save, then **Enter**.
3. Press **Ctrl + X** to exit.

This is where you'll often write:

- Bash scripts
- Configuration files
- Log notes

---

## View File Content

```bash
cat filename.txt     # Shows full content
less filename.txt    # Opens in scrollable view
head filename.txt    # Shows first 10 lines
tail filename.txt    # Shows last 10 lines
```

**Tail in action:**

```bash
tail -f /var/log/syslog
```

Live-streams system logs — very handy when debugging.

---

## Rename or Move a File/Folder

```bash
mv oldname.txt newname.txt
```

Renames the file.

You can also **move** it to another location:

```bash
mv notes.txt /home/nimesha/documents/
```

**Rename while moving:**

```bash
mv config.yml /etc/nginx/nginx.conf
```

---

## Copy a File or Folder

```bash
cp source.txt destination.txt
```

To copy a whole folder:

```bash
cp -r folder1 folder2
```

The `-r` means **recursive** — required for directories.

**Example:** You want to back up a config file before editing:

```bash
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

---

## Delete Files and Directories

```bash
rm file.txt
```

To remove a folder and its contents:

```bash
rm -r myfolder/
```

To force delete without confirmation:

```bash
rm -rf myfolder/
```

> ⚠️ **Be very careful with `rm -rf`.** It permanently deletes files — there's no undo!

---

## Pro Tip: Use `ls` After Every Action

Whenever you create, delete, or move something, run `ls` to verify:

```bash
ls -l
```

This builds your confidence and helps you avoid mistakes.

---

## Real-World Task

**1. Create a project directory:**

```bash
mkdir ~/devops/log-rotator
cd ~/devops/log-rotator
```

**2. Create and edit a file:**

```bash
touch rotate.sh
nano rotate.sh
```

**3. Back it up:**

```bash
cp rotate.sh rotate_backup.sh
```

**4. Move it to another folder:**

```bash
mv rotate_backup.sh ~/backups/
```

**5. Clean up:**

```bash
rm rotate.sh
```
