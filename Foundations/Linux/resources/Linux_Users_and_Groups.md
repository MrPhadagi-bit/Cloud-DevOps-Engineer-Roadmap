# Linux Users and Groups

> A comprehensive guide to understanding and managing users, groups, and permissions in Linux.
> 
> **Reference:** [Linux Users and Groups Tutorial](https://youtu.be/b-9j2jiCOEA?si=i5FEw-qaKElw-t8B)

---

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Understanding Linux Users](#2-understanding-linux-users)
  - [2.1 Types of Users](#21-types-of-users)
  - [2.2 User Identification (UID)](#22-user-identification-uid)
  - [2.3 User Properties](#23-user-properties)
- [3. Understanding Linux Groups](#3-understanding-linux-groups)
  - [3.1 Primary vs. Secondary Groups](#31-primary-vs-secondary-groups)
  - [3.2 Group Identification (GID)](#32-group-identification-gid)
  - [3.3 Built-in vs. Custom Groups](#33-built-in-vs-custom-groups)
- [4. Core System Files](#4-core-system-files)
  - [4.1 /etc/passwd](#41-etcpasswd)
  - [4.2 /etc/shadow](#42-etcshadow)
  - [4.3 /etc/group](#43-etcgroup)
- [5. File Permissions](#5-file-permissions)
  - [5.1 Permission Types](#51-permission-types)
  - [5.2 Permission Classes](#52-permission-classes)
  - [5.3 Numeric (Octal) Notation](#53-numeric-octal-notation)
  - [5.4 Symbolic Notation](#54-symbolic-notation)
  - [5.5 Default Permissions](#55-default-permissions)
- [6. User Management Commands](#6-user-management-commands)
  - [6.1 Creating Users](#61-creating-users)
  - [6.2 Modifying Users](#62-modifying-users)
  - [6.3 Deleting Users](#63-deleting-users)
  - [6.4 Password Management](#64-password-management)
- [7. Group Management Commands](#7-group-management-commands)
  - [7.1 Creating Groups](#71-creating-groups)
  - [7.2 Modifying Groups](#72-modifying-groups)
  - [7.3 Deleting Groups](#73-deleting-groups)
  - [7.4 Managing Group Members](#74-managing-group-members)
- [8. Permission Management Commands](#8-permission-management-commands)
  - [8.1 chmod - Change Mode](#81-chmod---change-mode)
  - [8.2 chown - Change Owner](#82-chown---change-owner)
  - [8.3 chgrp - Change Group](#83-chgrp---change-group)
- [9. Privilege Escalation](#9-privilege-escalation)
  - [9.1 su - Switch User](#91-su---switch-user)
  - [9.2 sudo - Superuser Do](#92-sudo---superuser-do)
  - [9.3 Configuring sudo](#93-configuring-sudo)
- [10. Best Practices](#10-best-practices)
- [11. Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Introduction

Linux is a **multi-user operating system** designed from the ground up to support multiple users simultaneously while maintaining strict security boundaries. Unlike single-user systems, Linux enforces a robust permission model that prevents users from accessing files, directories, and resources they are not authorized to use.

The Linux security model is built on three pillars:

1. **Users** – Individual accounts that identify who is using the system
2. **Groups** – Collections of users that share common access rights
3. **Permissions** – Rules that define what actions (read, write, execute) can be performed on files and directories

> **Key Principle:** Linux follows the **"least privilege"** principle—users and processes are granted only the minimum permissions necessary to perform their tasks.

---

## 2. Understanding Linux Users

A **user** in Linux represents an entity (human or system process) that can log in and interact with the system. Every user has a unique identity and a set of associated properties.

### 2.1 Types of Users

Linux categorizes users into three main types:

| Type | UID Range | Description |
|------|-----------|-------------|
| **Root (Superuser)** | `0` | The administrative account with unlimited privileges. Can modify any file, manage users, and change system configurations. |
| **System Users** | `1` – `999` | Accounts created for system services and daemons (e.g., `www-data`, `mysql`, `sshd`). These users cannot log in interactively and exist only to run specific services with restricted permissions. |
| **Regular Users** | `1000+` | Human users created by administrators. Each regular user has a home directory and can log in interactively. |

> **⚠️ Warning:** Operating as root is dangerous! You can accidentally delete critical system files or compromise security. Always use `sudo` for administrative tasks instead of logging in as root.

### 2.2 User Identification (UID)

Every user is assigned a unique **User ID (UID)**—a numeric identifier that the kernel uses internally to track ownership and permissions. While humans interact with usernames, the system operates using UIDs.

```bash
# View your current UID
echo $UID
# Output: 1000

# View all UIDs on the system
cat /etc/passwd | cut -d: -f1,3
```

**UID Ranges Explained:**

| Range | Purpose |
|-------|---------|
| `0` | Root user |
| `1` – `99` | Predefined system accounts |
| `100` – `999` | Dynamically allocated system accounts |
| `1000` – `60000` | Regular user accounts |
| `65534` | The `nobody` user (minimal privileges) |

### 2.3 User Properties

Each user account has the following properties:

| Property | Description | Example |
|----------|-------------|---------|
| **Username** | Human-readable login name | `john_doe` |
| **UID** | Unique numeric identifier | `1001` |
| **GID** | Primary group ID | `1001` |
| **Home Directory** | Personal workspace for the user | `/home/john_doe` |
| **Shell** | Default command interpreter | `/bin/bash` |
| **Password** | Encrypted authentication credential | Stored in `/etc/shadow` |
| **Comment/GECOS** | Full name or description | `John Doe, Developer` |

---

## 3. Understanding Linux Groups

A **group** is a collection of users who share common access rights to files and resources. Groups simplify permission management by allowing administrators to assign permissions to a group rather than individual users.

### 3.1 Primary vs. Secondary Groups

Every user in Linux belongs to at least one group, but can belong to multiple:

#### Primary Group
- **Definition:** The default group assigned to a user when their account is created.
- **Behavior:** When a user creates a file, the file's group ownership is automatically set to the user's primary group.
- **Limit:** Each user has exactly **one** primary group.

```bash
# When creating a user, a primary group with the same name is auto-created
sudo useradd --create-home alice
# This creates user 'alice' AND primary group 'alice'
```

#### Secondary (Supplementary) Groups
- **Definition:** Additional groups a user can be added to for extra permissions.
- **Behavior:** Users inherit all permissions granted to any of their secondary groups.
- **Limit:** A user can belong to up to **15** secondary groups.

```bash
# Add user 'alice' to secondary groups 'developers' and 'docker'
sudo usermod -aG developers,docker alice
```

### 3.2 Group Identification (GID)

Similar to UIDs, every group has a unique **Group ID (GID)**:

| Range | Purpose |
|-------|---------|
| `0` | Root group |
| `1` – `999` | System groups |
| `1000+` | Regular user groups |

```bash
# View all groups and their GIDs
cat /etc/group | cut -d: -f1,3

# Check which groups a user belongs to
groups alice
# Output: alice : alice developers docker
```

### 3.3 Built-in vs. Custom Groups

#### Built-in Groups
These are pre-installed groups essential for system operations:

| Group | Purpose |
|-------|---------|
| `root` | Full administrative access |
| `sudo` | Members can execute commands with superuser privileges |
| `users` | Default group for regular users (on some distributions) |
| `adm` | Access to system log files |
| `wheel` | Alternative administrative group (common on RHEL/CentOS) |

#### Custom Groups
Administrators create custom groups to organize users by role, department, or project:

```bash
# Create a group for the development team
sudo groupadd developers

# Create a group for the marketing team
sudo groupadd marketing
```

---

## 4. Core System Files

Linux stores user and group information in three critical files. Understanding their structure is essential for system administration.

### 4.1 /etc/passwd

The `/etc/passwd` file contains basic user account information. **Despite its name, it does NOT store passwords** (those moved to `/etc/shadow` for security).

```bash
# View the contents
cat /etc/passwd
```

**Format:**
```
username:password:UID:GID:GECOS:home_directory:shell
```

**Example Entry:**
```
alice:x:1001:1001:Alice Smith,Developer:/home/alice:/bin/bash
```

| Field | Value | Description |
|-------|-------|-------------|
| 1 | `alice` | Username |
| 2 | `x` | Password placeholder (`x` indicates password is in `/etc/shadow`) |
| 3 | `1001` | User ID (UID) |
| 4 | `1001` | Primary Group ID (GID) |
| 5 | `Alice Smith,Developer` | GECOS field (full name, contact info, etc.) |
| 6 | `/home/alice` | Home directory path |
| 7 | `/bin/bash` | Default login shell |

> **Security Note:** `/etc/passwd` is world-readable (`-rw-r--r--`) because many system utilities need to map UIDs to usernames. However, it does not contain actual password hashes.

### 4.2 /etc/shadow

The `/etc/shadow` file stores encrypted password hashes and password aging information. It is **only readable by root** (`-rw-r-----`).

```bash
# View with root privileges
sudo cat /etc/shadow
```

**Format:**
```
username:password_hash:last_changed:min_days:max_days:warn_days:inactive_days:expiry_date:reserved
```

**Example Entry:**
```
alice:$6$rounds=5000$saltsalt$hashhashhash:19000:0:99999:7:::
```

| Field | Description |
|-------|-------------|
| 1 | Username |
| 2 | Encrypted password hash (or `*` / `!` for locked accounts) |
| 3 | Days since epoch (Jan 1, 1970) when password was last changed |
| 4 | Minimum days before password can be changed |
| 5 | Maximum days before password must be changed |
| 6 | Days before expiry to warn the user |
| 7 | Days after expiry before account is disabled |
| 8 | Account expiration date (days since epoch) |
| 9 | Reserved field |

**Special Values in Password Field:**

| Value | Meaning |
|-------|---------|
| `*` | Account is disabled, no password can log in |
| `!` | Account is locked |
| Empty | No password required (dangerous!) |

```bash
# Change password aging information
sudo chage -l alice      # View current settings
sudo chage -M 90 alice   # Require password change every 90 days
```

### 4.3 /etc/group

The `/etc/group` file defines all groups on the system and their members.

```bash
cat /etc/group
```

**Format:**
```
group_name:password:GID:member1,member2,member3
```

**Example Entry:**
```
developers:x:1005:alice,bob,charlie
```

| Field | Description |
|-------|-------------|
| 1 | Group name |
| 2 | Group password (rarely used, usually `x`) |
| 3 | Group ID (GID) |
| 4 | Comma-separated list of member usernames |

> **Important:** The primary group membership is **NOT** listed in the member list of `/etc/group`. It is defined in `/etc/passwd` (field 4). Only secondary group memberships appear in `/etc/group`.

---

## 5. File Permissions

Linux file permissions control who can read, write, or execute files and directories. This is the core mechanism for enforcing security boundaries.

### 5.1 Permission Types

There are three basic permission types:

| Permission | Symbol | Numeric | File Effect | Directory Effect |
|------------|--------|---------|-------------|------------------|
| **Read** | `r` | `4` | View file contents | List directory contents (`ls`) |
| **Write** | `w` | `2` | Modify file contents | Create, delete, rename files |
| **Execute** | `x` | `1` | Run file as a program | Enter directory (`cd`) |

### 5.2 Permission Classes

Permissions are assigned to three classes of users:

| Class | Description |
|-------|-------------|
| **User (u)** | The owner of the file |
| **Group (g)** | Members of the file's group |
| **Others (o)** | Everyone else on the system |

**Viewing Permissions:**

```bash
ls -l file.txt
# Output: -rw-r--r-- 1 alice developers 1234 Aug 29 10:00 file.txt
```

**Permission String Breakdown:**
```
-rw-r--r--
|  |  |
|  |  └── Others: r-- (read only)
|  └─────── Group: r-- (read only)
└────────── User:  rw- (read + write)
```

| Position | Meaning |
|----------|---------|
| 1 | File type (`-` = file, `d` = directory, `l` = symlink) |
| 2-4 | User (owner) permissions |
| 5-7 | Group permissions |
| 8-10 | Others permissions |

### 5.3 Numeric (Octal) Notation

Permissions can be represented as a 3-digit octal number, where each digit is the sum of permission values:

```
Read (4) + Write (2) + Execute (1) = Permission Value
```

**Common Permission Combinations:**

| Numeric | Symbolic | Meaning |
|---------|----------|---------|
| `7` | `rwx` | Read + Write + Execute |
| `6` | `rw-` | Read + Write |
| `5` | `r-x` | Read + Execute |
| `4` | `r--` | Read only |
| `0` | `---` | No permissions |

**Full Examples:**

| Numeric | User | Group | Others | Use Case |
|---------|------|-------|--------|----------|
| `777` | `rwx` | `rwx` | `rwx` | Full access to everyone (⚠️ dangerous) |
| `755` | `rwx` | `r-x` | `r-x` | Executable file/directory, others can read |
| `644` | `rw-` | `r--` | `r--` | Regular file, owner can edit, others read |
| `700` | `rwx` | `---` | `---` | Private file/directory, only owner access |
| `750` | `rwx` | `r-x` | `---` | Owner full, group read/execute, others none |

```bash
# Set permissions using numeric mode
chmod 644 file.txt    # rw-r--r--
chmod 755 script.sh   # rwxr-xr-x
chmod 700 private/    # rwx------
```

### 5.4 Symbolic Notation

Symbolic mode uses letters and operators to modify permissions:

**Operators:**

| Operator | Meaning |
|----------|---------|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

**Examples:**

```bash
# Add execute permission for the user
chmod u+x script.sh

# Remove write permission for others
chmod o-w file.txt

# Set group to read and write only
chmod g=rw file.txt

# Add read permission for everyone
chmod a+r file.txt

# Remove all permissions for others
chmod o= file.txt
```

### 5.5 Default Permissions

When files and directories are created, they receive default permissions based on the **umask** value.

```bash
# View current umask
umask
# Output: 0022
```

**How umask Works:**
- Base permissions for files: `666` (rw-rw-rw-)
- Base permissions for directories: `777` (rwxrwxrwx)
- Subtract umask to get final permissions

**Example with umask `0022`:**

| Type | Base | Umask | Result | Symbolic |
|------|------|-------|--------|----------|
| File | `666` | `022` | `644` | `rw-r--r--` |
| Directory | `777` | `022` | `755` | `rwxr-xr-x` |

```bash
# Set a more restrictive umask (private by default)
umask 0077
# Now new files: 600 (rw-------)
# Now new directories: 700 (rwx------)
```

---

## 6. User Management Commands

### 6.1 Creating Users

#### `useradd` – Create a New User

```bash
# Basic user creation
sudo useradd username

# Create user with home directory
sudo useradd --create-home username
# or
sudo useradd -m username

# Create user with specific home directory
sudo useradd -m -d /custom/home/dir username

# Create user with specific shell
sudo useradd -m -s /bin/zsh username

# Create user with comment/description
sudo useradd -m -c "John Doe, Developer" username

# Create user with specific UID
sudo useradd -m -u 1500 username

# Create user with specific primary group
sudo useradd -m -g developers username

# Create user with secondary groups
sudo useradd -m -G sudo,docker,developers username

# Create a system user (no login, no home)
sudo useradd -r -s /usr/sbin/nologin serviceaccount
```

**Common `useradd` Options:**

| Option | Description |
|--------|-------------|
| `-m`, `--create-home` | Create home directory |
| `-d`, `--home` | Specify home directory path |
| `-s`, `--shell` | Specify login shell |
| `-c`, `--comment` | Add GECOS comment |
| `-u`, `--uid` | Specify UID |
| `-g`, `--gid` | Specify primary group (by name or GID) |
| `-G`, `--groups` | Specify secondary groups (comma-separated) |
| `-r`, `--system` | Create a system account |
| `-e`, `--expiredate` | Set account expiration date |
| `-f`, `--inactive` | Set password inactivity period |

#### `adduser` – Interactive Alternative (Debian/Ubuntu)

```bash
# More user-friendly, interactive prompt
sudo adduser username
# Prompts for password, full name, and other information
```

### 6.2 Modifying Users

#### `usermod` – Modify a User Account

```bash
# Change user's login name
sudo usermod -l newname oldname

# Change user's home directory
sudo usermod -d /new/home/dir username

# Change user's shell
sudo usermod -s /bin/zsh username

# Change user's UID
sudo usermod -u 2000 username

# Change user's primary group
sudo usermod -g newgroup username

# Add user to secondary groups (REPLACES existing groups!)
sudo usermod -G group1,group2 username

# Append user to secondary groups (preserves existing)
sudo usermod -aG group1,group2 username
# ⚠️ Always use -a (append) with -G to avoid removing existing groups!

# Lock a user account
sudo usermod -L username

# Unlock a user account
sudo usermod -U username

# Set account expiration date
sudo usermod -e 2025-12-31 username
```

### 6.3 Deleting Users

#### `userdel` – Delete a User Account

```bash
# Delete user (keeps home directory and mail spool)
sudo userdel username

# Delete user and remove home directory
sudo userdel -r username
# or
sudo userdel --remove username

# Delete user and remove home directory AND mail spool
sudo userdel -r --remove-all-files username
```

> **⚠️ Warning:** `userdel -r` is irreversible. Always back up important data before removing a user.

### 6.4 Password Management

#### `passwd` – Change Passwords

```bash
# Change your own password
passwd

# Change another user's password (as root)
sudo passwd username

# Lock a user's password
sudo passwd -l username

# Unlock a user's password
sudo passwd -u username

# Force user to change password at next login
sudo passwd --expire username

# Display password status
sudo passwd -S username
```

#### `chage` – Manage Password Aging

```bash
# View password aging information
sudo chage -l username

# Set minimum password age (days before change allowed)
sudo chage -m 7 username

# Set maximum password age (days before change required)
sudo chage -M 90 username

# Set password expiration warning period
sudo chage -W 14 username

# Set account expiration date
sudo chage -E 2025-12-31 username

# Interactive mode
sudo chage username
```

---

## 7. Group Management Commands

### 7.1 Creating Groups

#### `groupadd` – Create a New Group

```bash
# Create a new group
sudo groupadd developers

# Create group with specific GID
sudo groupadd -g 2000 developers

# Create a system group
sudo groupadd -r sysgroup
```

### 7.2 Modifying Groups

#### `groupmod` – Modify a Group

```bash
# Change group name
sudo groupmod -n newname oldname

# Change group GID
sudo groupmod -g 3000 groupname
```

### 7.3 Deleting Groups

#### `groupdel` – Delete a Group

```bash
# Delete a group
sudo groupdel groupname
```

> **⚠️ Note:** You cannot delete a group that is a user's primary group. Change the user's primary group first.

### 7.4 Managing Group Members

#### `gpasswd` – Manage Group Members

```bash
# Add a user to a group
sudo gpasswd -a username groupname

# Remove a user from a group
sudo gpasswd -d username groupname

# Set group administrators
sudo gpasswd -A admin1,admin2 groupname

# Set group members
sudo gpasswd -M user1,user2,user3 groupname

# Remove group password
sudo gpasswd -r groupname
```

#### Alternative: `usermod` for Group Membership

```bash
# Add user to group (append mode)
sudo usermod -aG groupname username

# Remove user from ALL secondary groups except specified
sudo usermod -G groupname username
```

#### Checking Group Membership

```bash
# Check which groups a user belongs to
groups username

# Check which users belong to a group
grep groupname /etc/group

# Get detailed group information
getent group groupname
```

---

## 8. Permission Management Commands

### 8.1 chmod – Change Mode (Permissions)

The `chmod` command changes the permissions of files and directories.

**Numeric Mode:**

```bash
# Set exact permissions
chmod 644 file.txt       # rw-r--r--
chmod 755 directory/     # rwxr-xr-x
chmod 700 secret.key     # rwx------

# Recursive permission change
chmod -R 755 project/

# Apply permissions to all files in a directory
find . -type f -exec chmod 644 {} +
find . -type d -exec chmod 755 {} +
```

**Symbolic Mode:**

```bash
# Add execute for user
chmod u+x script.sh

# Remove write for group and others
chmod go-w file.txt

# Set exact permissions for user
chmod u=rw file.txt

# Add read for all (user, group, others)
chmod a+r file.txt
# or
chmod +r file.txt

# Recursive symbolic change
chmod -R g+w shared/
```

### 8.2 chown – Change Owner

The `chown` command changes the user and/or group ownership of files.

```bash
# Change file owner
sudo chown newuser file.txt

# Change file owner and group
sudo chown newuser:newgroup file.txt

# Change only the group
sudo chown :newgroup file.txt
# or use chgrp
sudo chgrp newgroup file.txt

# Recursive ownership change
sudo chown -R newuser:newgroup directory/

# Change ownership based on reference file
sudo chown --reference=otherfile.txt targetfile.txt

# Change ownership of a symbolic link itself (not target)
sudo chown -h user:group symlink
```

### 8.3 chgrp – Change Group

```bash
# Change group ownership
sudo chgrp developers file.txt

# Recursive group change
sudo chgrp -R developers project/
```

---

## 9. Privilege Escalation

### 9.1 su – Switch User

The `su` command switches to another user account.

```bash
# Switch to root (requires root password)
su

# Switch to root with root's environment
su -
# or
su -l
# The dash (-) provides a login environment with the target user's PATH, variables, etc.

# Switch to another user
su username

# Switch to another user with their environment
su - username

# Execute a single command as another user
su -c "command" username
```

### 9.2 sudo – Superuser Do

The `sudo` command allows authorized users to execute commands as root or another user.

```bash
# Execute command as root
sudo command

# Execute command as another user
sudo -u username command

# Open a root shell
sudo -i

# Open a root shell with root's environment
sudo -s

# Edit a file as root
sudo nano /etc/hosts

# Execute command with root's environment
sudo -E command

# List allowed commands for current user
sudo -l

# Run command without password (if configured)
sudo -n command
```

### 9.3 Configuring sudo

The `sudoers` file controls who can use `sudo` and what they can do.

> **⚠️ CRITICAL:** Always edit `/etc/sudoers` using `visudo` to prevent syntax errors that could lock you out of root access!

```bash
# Safely edit sudoers file
sudo visudo
```

**Basic sudoers Syntax:**

```
# User privilege specification
username    ALL=(ALL:ALL) ALL

# Group privilege specification
%groupname  ALL=(ALL:ALL) ALL

# Allow specific command without password
username    ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Allow user to run commands as specific user
username    ALL=(www-data) ALL

# Restrict to specific host
username    hostname=(ALL) ALL
```

**sudoers Fields Explained:**

```
user    host=(runas:runas_group) command
```

| Field | Description |
|-------|-------------|
| `user` | Who is allowed (username or `%groupname`) |
| `host` | Which hosts (usually `ALL`) |
| `runas` | User to run as (usually `ALL` for any user, or `root`) |
| `runas_group` | Group to run as |
| `command` | Allowed commands (use `ALL` for any, or specify path) |

**Common sudoers Aliases:**

```
# User aliases
User_Alias ADMINS = alice, bob, charlie

# Command aliases
Cmnd_Alias SOFTWARE = /usr/bin/apt, /usr/bin/dpkg
Cmnd_Alias SERVICES = /usr/bin/systemctl *

# Runas aliases
Runas_Alias WEB = www-data, nginx

# Host aliases
Host_Alias LOCAL = localhost, 192.168.1.0/24

# Grant privileges
ADMINS LOCAL=(ALL) NOPASSWD: SOFTWARE
```

---

## 10. Best Practices

### Security Principles

1. **Least Privilege Principle**
   - Grant users only the permissions they absolutely need
   - Use groups to manage permissions rather than individual user assignments
   - Regularly audit user permissions and group memberships

2. **Avoid Root Login**
   - Never log in directly as root
   - Use `sudo` for administrative tasks
   - Disable root login via SSH (`PermitRootLogin no` in `/etc/ssh/sshd_config`)

3. **Strong Password Policy**
   - Enforce minimum password length and complexity
   - Set password expiration using `chage`
   - Use SSH keys instead of passwords where possible

4. **Regular Auditing**
   ```bash
   # List all user accounts
   cat /etc/passwd | grep -E ":(1000|500):"  # Regular users

   # Check for accounts without passwords
   sudo awk -F: '($2 == "") {print $1}' /etc/shadow

   # Check for locked accounts
   sudo passwd -S -a | grep L

   # Review sudo access
   sudo cat /etc/sudoers
   sudo ls /etc/sudoers.d/
   ```

### Group Management Best Practices

1. **Organize by Role/Function**
   - Create groups based on teams (e.g., `developers`, `qa`, `devops`)
   - Create groups based on access needs (e.g., `docker`, `sudo`, `vpn`)

2. **Use Secondary Groups for Flexibility**
   - Keep primary groups simple (usually same as username)
   - Add users to multiple secondary groups for different access levels

3. **Document Group Purposes**
   - Maintain documentation about what each group is for
   - Remove unused groups periodically

### Permission Management Best Practices

1. **Never Use 777**
   - `777` (rwxrwxrwx) gives full access to everyone—avoid it
   - Use `755` for directories and `644` for files when sharing is needed
   - Use `750` / `640` for sensitive data

2. **Set Proper Umask**
   - Default `0022` is reasonable for most servers
   - Use `0077` for highly sensitive environments

3. **Verify Before Recursive Changes**
   ```bash
   # Preview what chmod -R would do (dry run)
   find . -type f | head

   # Test on a single file first
   chmod 600 testfile.txt
   ls -l testfile.txt
   ```

---

## 11. Quick Reference Cheat Sheet

### User Commands

| Command | Description |
|---------|-------------|
| `useradd -m username` | Create user with home directory |
| `userdel -r username` | Delete user and home directory |
| `usermod -aG group user` | Add user to group |
| `usermod -g group user` | Change primary group |
| `passwd username` | Change password |
| `chage -l username` | View password aging info |

### Group Commands

| Command | Description |
|---------|-------------|
| `groupadd groupname` | Create group |
| `groupdel groupname` | Delete group |
| `groupmod -n new old` | Rename group |
| `gpasswd -a user group` | Add user to group |
| `gpasswd -d user group` | Remove user from group |
| `groups username` | List user's groups |

### Permission Commands

| Command | Description |
|---------|-------------|
| `chmod 755 file` | Set numeric permissions |
| `chmod u+x file` | Add execute for user |
| `chown user:group file` | Change owner and group |
| `chgrp group file` | Change group only |
| `umask 0022` | Set default permission mask |

### Privilege Commands

| Command | Description |
|---------|-------------|
| `sudo command` | Run as root |
| `sudo -i` | Root interactive shell |
| `su - username` | Switch to user |
| `visudo` | Edit sudoers safely |

### Common Permission Patterns

| Pattern | Files | Directories |
|---------|-------|-------------|
| Private | `600` (`rw-------`) | `700` (`rwx------`) |
| Shared (group) | `640` (`rw-r-----`) | `750` (`rwxr-x---`) |
| Public | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`) |
| Executable | `755` (`rwxr-xr-x`) | `755` (`rwxr-xr-x`) |

---

## Additional Resources

- [Linux Foundation - User and Group Management](https://www.linuxfoundation.org/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- `man passwd`, `man group`, `man chmod`, `man sudoers`

---

*This guide was created as a comprehensive reference for Linux user and group management. For the video tutorial that inspired this documentation, see the reference link at the top of this document.*

**License:** MIT — Feel free to use, modify, and distribute this guide.
