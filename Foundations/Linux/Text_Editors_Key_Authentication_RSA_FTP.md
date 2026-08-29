# Text Editors: Key Authentication (RSA & FTP)

> A comprehensive guide to understanding and implementing RSA key authentication and FTP/SFTP key-based authentication when working with text editors, IDEs, and remote development environments.
>
> **Reference Video:** [YouTube — Text Editors: Key Authentication](https://youtu.be/8Mt7SH2Voi0?si=Xq18AweCtW8-XGll)

---

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Understanding Key Authentication](#2-understanding-key-authentication)
  - [2.1 What is Public Key Cryptography?](#21-what-is-public-key-cryptography)
  - [2.2 How RSA Key Authentication Works](#22-how-rsa-key-authentication-works)
  - [2.3 FTP vs. SFTP vs. FTPS](#23-ftp-vs-sftp-vs-ftps)
- [3. Generating RSA Key Pairs](#3-generating-rsa-key-pairs)
  - [3.1 Using OpenSSH (Linux/macOS/Git Bash)](#31-using-openssh-linuxmacosgit-bash)
  - [3.2 Using PuTTYgen (Windows)](#32-using-puttygen-windows)
- [4. Configuring Key Authentication for Text Editors](#4-configuring-key-authentication-for-text-editors)
  - [4.1 Visual Studio Code (VS Code)](#41-visual-studio-code-vs-code)
  - [4.2 Sublime Text](#42-sublime-text)
  - [4.3 Vim / Neovim](#43-vim--neovim)
  - [4.4 Emacs](#44-emacs)
  - [4.5 JetBrains IDEs](#45-jetbrains-ides)
- [5. Setting Up SFTP Key Authentication](#5-setting-up-sftp-key-authentication)
  - [5.1 Server-Side Configuration](#51-server-side-configuration)
  - [5.2 Client-Side Configuration](#52-client-side-configuration)
  - [5.3 Using ssh-copy-id](#53-using-ssh-copy-id)
- [6. Common Text Editor SFTP Plugins & Extensions](#6-common-text-editor-sftp-plugins--extensions)
- [7. Troubleshooting](#7-troubleshooting)
- [8. Security Best Practices](#8-security-best-practices)
- [9. Glossary](#9-glossary)
- [10. Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. Introduction

When working with remote servers, cloud instances, or shared development environments, text editors and IDEs often need to connect securely to transfer files or execute remote commands. **Key authentication** provides a more secure and convenient alternative to password-based logins.

This guide covers:
- The fundamentals of **RSA (Rivest–Shamir–Adleman)** public-key cryptography
- How to generate, configure, and manage SSH key pairs
- Setting up **SFTP (SSH File Transfer Protocol)** with key authentication
- Integrating key authentication into popular **text editors and IDEs**

---

## 2. Understanding Key Authentication

### 2.1 What is Public Key Cryptography?

Public key cryptography uses a pair of mathematically linked keys:

| Key Type | Description | Location |
|----------|-------------|----------|
| **Public Key** | Shared openly; acts like a lock | Stored on the remote server (`~/.ssh/authorized_keys`) |
| **Private Key** | Kept secret; acts like the key | Stored securely on your local machine (`~/.ssh/id_rsa`) |
| **Passphrase** | Optional password to encrypt the private key | Memorized by the user |

> **Core Principle:** Data encrypted with the public key can *only* be decrypted with the private key, and vice versa. This ensures that even if someone intercepts the public key, they cannot forge authentication.

### 2.2 How RSA Key Authentication Works

The RSA authentication process follows these steps:

```
┌─────────────────┐                      ┌─────────────────┐
│   SFTP Client   │                      │   SFTP Server   │
│  (Your Machine) │                      │  (Remote Host)  │
└────────┬────────┘                      └────────┬────────┘
         │                                        │
         │  1. Client initiates connection        │
         │───────────────────────────────────────>│
         │                                        │
         │  2. Server sends challenge (random data)│
         │<───────────────────────────────────────│
         │                                        │
         │  3. Client signs challenge with        │
         │     PRIVATE KEY                        │
         │                                        │
         │  4. Client sends signed challenge      │
         │     + PUBLIC KEY                       │
         │───────────────────────────────────────>│
         │                                        │
         │  5. Server verifies signature using    │
         │     stored PUBLIC KEY                  │
         │     (in authorized_keys)               │
         │                                        │
         │  6. Server grants access               │
         │<───────────────────────────────────────│
         │                                        │
```

**Key Points:**
- The **private key never leaves your machine** — this is what makes it secure.
- The server only needs your **public key** to verify your identity.
- A **passphrase** adds an extra layer of security to your private key.

### 2.3 FTP vs. SFTP vs. FTPS

Understanding the differences is critical when configuring your text editor:

| Protocol | Encryption | Authentication | Port | Use Case |
|----------|------------|----------------|------|----------|
| **FTP** | ❌ None (plaintext) | Password only | 21 | ⚠️ Legacy — avoid for sensitive data |
| **SFTP** | ✅ SSH encryption | Password or SSH Keys | 22 | ✅ Recommended — secure file transfer over SSH |
| **FTPS** | ✅ TLS/SSL encryption | Password or Certificates | 21/990 | ✅ Secure, but more complex firewall rules |

> **Recommendation:** Always prefer **SFTP** over FTP. SFTP uses the SSH protocol for both authentication and encryption, making it inherently more secure and easier to configure with key pairs.

---

## 3. Generating RSA Key Pairs

### 3.1 Using OpenSSH (Linux / macOS / Git Bash on Windows)

OpenSSH is the standard tool for generating and managing SSH keys. It comes pre-installed on most Unix-like systems.

#### Step 1: Generate the Key Pair

```bash
# Generate a 4096-bit RSA key pair (recommended)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Parameter Breakdown:**
| Flag | Description |
|------|-------------|
| `-t rsa` | Specifies RSA as the key type |
| `-b 4096` | Sets the key size to 4096 bits (more secure than default 3072) |
| `-C "comment"` | Adds a label (usually your email) to identify the key |

#### Step 2: Respond to Prompts

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): [Press ENTER for default]
Enter passphrase (empty for no passphrase): [Type a strong passphrase]
Enter same passphrase again: [Confirm passphrase]
Your identification has been saved in /home/username/.ssh/id_rsa
Your public key has been saved in /home/username/.ssh/id_rsa.pub
```

#### Step 3: Verify the Keys

```bash
# List your SSH directory
ls -la ~/.ssh/

# Expected output:
# id_rsa      ← Your PRIVATE key (keep secret!)
# id_rsa.pub  ← Your PUBLIC key (share with servers)
```

#### Step 4: Set Proper Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

> **Security Note:** SSH is strict about permissions. If `~/.ssh` or `id_rsa` are too permissive, SSH will refuse to use the key.

### 3.2 Using PuTTYgen (Windows)

For Windows users who prefer a GUI or need `.ppk` format keys for PuTTY-based tools:

#### Step 1: Download and Launch PuTTYgen

1. Download PuTTYgen from the [official PuTTY website](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).
2. Launch `puttygen.exe`.

#### Step 2: Generate the Key

1. Ensure **RSA** is selected at the bottom of the window.
2. Change the number of bits to **4096**.
3. Click **Generate**.
4. Move your mouse randomly over the gray area to generate entropy.

#### Step 3: Save the Keys

1. **Update the key comment** (e.g., `username@hostname`).
2. **Copy the public key** from the text box — this is what goes into `authorized_keys`.
3. **Enter a passphrase** (optional but highly recommended).
4. Click **Save private key** → save as `id_rsa.ppk`.
5. Save the public key separately as `id_rsa.pub` for convenience.

#### Step 4: Convert to OpenSSH Format (if needed)

Some text editors require OpenSSH format instead of `.ppk`:

```
In PuTTYgen:
  Conversions → Export OpenSSH key → Save as id_rsa
```

---

## 4. Configuring Key Authentication for Text Editors

### 4.1 Visual Studio Code (VS Code)

VS Code supports remote development via the **Remote - SSH** extension and SFTP via extensions like **SFTP** (by liximomo).

#### Method A: Remote - SSH Extension (Recommended)

1. Install the **Remote - SSH** extension from Microsoft.
2. Press `F1` → type `Remote-SSH: Connect to Host...`
3. Enter: `ssh username@hostname`
4. VS Code will use your `~/.ssh/id_rsa` automatically if it's in the default location.

#### Method B: Configuring `~/.ssh/config`

Create or edit `~/.ssh/config`:

```ssh
Host myserver
    HostName 192.168.1.100
    User myusername
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

Then connect via: `F1` → `Remote-SSH: Connect to Host...` → select `myserver`.

#### Method C: SFTP Extension for File Sync

1. Install the **SFTP** extension by liximomo.
2. Press `F1` → `SFTP: Config`.
3. Edit `.vscode/sftp.json`:

```json
{
    "name": "My Server",
    "host": "192.168.1.100",
    "protocol": "sftp",
    "port": 22,
    "username": "myusername",
    "privateKeyPath": "~/.ssh/id_rsa",
    "passphrase": true,
    "remotePath": "/var/www/html",
    "uploadOnSave": true
}
```

> **Note:** Set `"passphrase": true` to be prompted for your key passphrase. Never hardcode passwords in config files.

### 4.2 Sublime Text

Sublime Text requires the **SFTP** package (by wbond) for remote file editing.

#### Setup Steps:

1. Install **Package Control** if not already installed.
2. `Ctrl+Shift+P` → `Package Control: Install Package` → search for **SFTP**.
3. `File` → `SFTP/FTP` → `Setup Server...`
4. Edit the generated `.sftp-config.json`:

```json
{
    "type": "sftp",
    "sync_down_on_open": true,
    "host": "192.168.1.100",
    "user": "myusername",
    "port": "22",
    "ssh_key_file": "~/.ssh/id_rsa",
    "ssh_key_file_passphrase": "",
    "remote_path": "/var/www/html",
    "connect_timeout": 30
}
```

5. Right-click in the sidebar → `SFTP/FTP` → `Browse Server...`

### 4.3 Vim / Neovim

Vim and Neovim can edit remote files natively using the `netrw` plugin (built-in).

#### Direct Remote Editing

```vim
" Edit a remote file directly
:e scp://username@hostname//path/to/file.txt

" Or with a specific key
:e scp://username@hostname//path/to/file.txt
```

#### Using `~/.ssh/config`

Vim respects your SSH config. Define a host:

```ssh
Host webserver
    HostName 192.168.1.100
    User myuser
    IdentityFile ~/.ssh/id_rsa
```

Then in Vim:

```vim
:e scp://webserver//var/www/index.html
```

#### Using Plugins (Neovim + Telescope)

For a more modern workflow, use **telescope.nvim** with **nvim-scp**:

```lua
-- In your init.lua or plugin config
require('telescope').setup({
    extensions = {
        scp = {
            hosts = {
                { name = "Production", host = "webserver", path = "/var/www" },
            }
        }
    }
})
```

### 4.4 Emacs

Emacs supports remote files via **TRAMP** (Transparent Remote Access, Multiple Protocols).

#### Basic TRAMP Syntax

```emacs
" Open a remote file
C-x C-f /ssh:username@hostname:/path/to/file.txt

" Or with a specific method
C-x C-f /scp:username@hostname:/path/to/file.txt

" With an explicit key file
C-x C-f /ssh:username@hostname#~/.ssh/id_rsa:/path/to/file.txt
```

#### Configuring SSH Keys in Emacs

TRAMP uses your system's SSH configuration. Ensure your `~/.ssh/config` is set up:

```ssh
Host myserver
    HostName 192.168.1.100
    User myuser
    IdentityFile ~/.ssh/id_rsa
```

Then simply use:

```emacs
C-x C-f /ssh:myserver:/var/www/index.html
```

#### Dired for Remote Directory Browsing

```emacs
" Browse remote directory
C-x d /ssh:myserver:/var/www/
```

### 4.5 JetBrains IDEs (IntelliJ, PyCharm, WebStorm, etc.)

JetBrains IDEs have built-in remote development and deployment tools.

#### Setting Up an SFTP Server

1. `Tools` → `Deployment` → `Configuration...`
2. Click `+` to add a new server → select **SFTP**.
3. Enter:
   - **Host:** `192.168.1.100`
   - **Port:** `22`
   - **Root path:** `/var/www/html`
   - **User name:** `myusername`
   - **Auth type:** `Key pair (OpenSSH or PuTTY)`
   - **Private key file:** Browse to `~/.ssh/id_rsa`
   - **Passphrase:** Enter if your key is encrypted.

4. Click **Test Connection** to verify.

#### Mappings

Set up path mappings so your local project syncs with the remote server:

```
Local path:   /home/user/projects/myapp/
Remote path:  /var/www/html/myapp/
```

#### Automatic Upload

Enable `Tools` → `Deployment` → `Automatic Upload` to sync files on save.

---

## 5. Setting Up SFTP Key Authentication

### 5.1 Server-Side Configuration

To enable key authentication on your SFTP server, you need to place your **public key** in the correct location.

#### Step 1: Create the `.ssh` Directory

Log into your server (via password initially) and run:

```bash
# Create .ssh directory in your home folder
mkdir -p ~/.ssh

# Set strict permissions
chmod 700 ~/.ssh
```

#### Step 2: Add Your Public Key to `authorized_keys`

```bash
# Create or edit the authorized_keys file
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Add your public key (copy from your local machine's id_rsa.pub)
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC... user@host" >> ~/.ssh/authorized_keys
```

> **Important:** The entire public key must be on **one line**. Do not add line breaks within the key string.

### 5.2 Client-Side Configuration

Ensure your local machine has the correct setup:

```bash
# Start the SSH agent (if not running)
eval "$(ssh-agent -s)"

# Add your private key to the agent
ssh-add ~/.ssh/id_rsa

# Enter your passphrase when prompted
```

To make this persistent on macOS, add to `~/.zshrc`:

```bash
# macOS: Add SSH keys to Apple Keychain
ssh-add --apple-use-keychain ~/.ssh/id_rsa
```

On Linux, the SSH agent usually starts automatically with your desktop session.

### 5.3 Using `ssh-copy-id`

The easiest way to copy your public key to a server is using `ssh-copy-id`:

```bash
# Automatically copies your public key to the server's authorized_keys
ssh-copy-id -i ~/.ssh/id_rsa.pub username@hostname

# Example
ssh-copy-id -i ~/.ssh/id_rsa.pub deploy@192.168.1.100
```

**What it does:**
1. Logs into the server using your password.
2. Creates `~/.ssh` and `~/.ssh/authorized_keys` if they don't exist.
3. Appends your public key to `authorized_keys`.
4. Sets the correct permissions (`700` for `.ssh`, `600` for `authorized_keys`).

#### Manual Copy Method

If `ssh-copy-id` is not available, manually copy the key:

```bash
# On your local machine, display the public key
cat ~/.ssh/id_rsa.pub

# Copy the output, then on the server:
echo "PASTE_KEY_HERE" >> ~/.ssh/authorized_keys
```

---

## 6. Common Text Editor SFTP Plugins & Extensions

| Text Editor / IDE | Plugin / Extension | Protocol | Key Support |
|-------------------|-------------------|----------|-------------|
| **VS Code** | Remote - SSH (Microsoft) | SSH/SFTP | ✅ OpenSSH, PuTTY |
| **VS Code** | SFTP (liximomo) | SFTP | ✅ OpenSSH keys |
| **Sublime Text** | SFTP (wbond) | SFTP/FTPS | ✅ OpenSSH, PuTTY |
| **Atom** | remote-ftp | SFTP | ✅ OpenSSH keys |
| **Vim/Neovim** | netrw (built-in) | SCP/SFTP | ✅ System SSH config |
| **Vim/Neovim** | nvim-scp | SCP | ✅ OpenSSH keys |
| **Emacs** | TRAMP (built-in) | SSH/SCP/SFTP | ✅ System SSH config |
| **JetBrains** | Built-in Deployment | SFTP/FTPS | ✅ OpenSSH, PuTTY |
| **Notepad++** | NppFTP | SFTP/FTPS | ✅ OpenSSH keys |
| **Kate** | Built-in Network | SFTP | ✅ System SSH config |

---

## 7. Troubleshooting

### Issue: "Permissions are too open" Error

**Symptom:**
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/user/.ssh/id_rsa' are too open.
```

**Fix:**
```bash
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh
```

### Issue: "Permission denied (publickey)"

**Symptom:** SSH/SFTP connection fails with a public key error.

**Checklist:**
1. ✅ Is the public key correctly added to `~/.ssh/authorized_keys` on the server?
2. ✅ Are server permissions correct? (`700` for `.ssh`, `600` for `authorized_keys`)
3. ✅ Is the private key path correct in your editor/IDE config?
4. ✅ Did you enter the correct passphrase?
5. ✅ Is the SSH service running on the server? (`sudo systemctl status sshd`)

### Issue: Text Editor Can't Find Private Key

**Symptom:** Editor says key file not found.

**Fix:**
- Use the **absolute path** instead of `~` (e.g., `/home/username/.ssh/id_rsa` instead of `~/.ssh/id_rsa`).
- On Windows, use forward slashes or double backslashes: `C:/Users/username/.ssh/id_rsa`.

### Issue: Passphrase Prompt Keeps Appearing

**Fix:**
```bash
# Add key to SSH agent
ssh-add ~/.ssh/id_rsa

# On macOS, add to Keychain for persistence
ssh-add -K ~/.ssh/id_rsa
```

### Issue: PuTTY Key Format Not Accepted

**Symptom:** Editor expects OpenSSH format but you have a `.ppk` file.

**Fix:**
1. Open the `.ppk` in PuTTYgen.
2. Go to **Conversions** → **Export OpenSSH key**.
3. Save as `id_rsa` (no extension) and use that file.

---

## 8. Security Best Practices

### 🔐 Key Generation
- **Always use 4096-bit RSA keys** (or Ed25519 for modern systems).
- **Always set a passphrase** on your private key.
- **Never share your private key** — treat it like a password.

### 🔐 Key Storage
- Store private keys in `~/.ssh/` with permissions `600`.
- Use an **SSH agent** to avoid typing passphrases repeatedly.
- On macOS, use the **Keychain** integration (`ssh-add -K`).
- On Windows, use **Pageant** (PuTTY's SSH agent) if using PuTTY tools.

### 🔐 Server Configuration
- Disable password authentication once key auth is working:
  ```bash
  # In /etc/ssh/sshd_config
  PasswordAuthentication no
  PubkeyAuthentication yes
  ChallengeResponseAuthentication no
  ```
- Restart SSH service after changes:
  ```bash
  sudo systemctl restart sshd   # Linux
  sudo service ssh restart      # macOS / older Linux
  ```

### 🔐 Regular Maintenance
- Rotate keys periodically (e.g., every 6–12 months).
- Remove old or unused keys from `authorized_keys`:
  ```bash
  # List all keys in authorized_keys
cat ~/.ssh/authorized_keys

# Remove a specific key by editing the file
nano ~/.ssh/authorized_keys
  ```
- Use separate keys for different servers/environments (dev, staging, production).

---

## 9. Glossary

| Term | Definition |
|------|------------|
| **RSA** | Rivest–Shamir–Adleman — a widely used public-key cryptosystem. |
| **SSH** | Secure Shell — a protocol for secure remote login and command execution. |
| **SFTP** | SSH File Transfer Protocol — secure file transfer over SSH (not related to FTP). |
| **FTPS** | FTP over SSL/TLS — traditional FTP with an added encryption layer. |
| **Public Key** | The shareable key used to verify identity; stored on servers. |
| **Private Key** | The secret key used to prove identity; kept on your local machine. |
| **Passphrase** | A password used to encrypt/decrypt the private key file. |
| **Authorized Keys** | The file (`~/.ssh/authorized_keys`) on a server that stores trusted public keys. |
| **Known Hosts** | The file (`~/.ssh/known_hosts`) storing fingerprints of servers you've connected to. |
| **SSH Agent** | A background program that holds private keys in memory to avoid repeated passphrase entry. |
| **Fingerprint** | A short hash representing a key or host identity, used for verification. |
| **Entropy** | Randomness collected during key generation to ensure unpredictability. |

---

## 10. Quick Reference Cheat Sheet

### Generate a New Key Pair
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Copy Public Key to Server
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@hostname
```

### Connect via SFTP
```bash
sftp -i ~/.ssh/id_rsa user@hostname
```

### Start SSH Agent & Add Key
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

### Test SSH Connection
```bash
ssh -i ~/.ssh/id_rsa user@hostname
```

### Fix Permissions
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys
```

### VS Code Remote SSH Config (`~/.ssh/config`)
```ssh
Host myserver
    HostName 192.168.1.100
    User myuser
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

---

## References & Further Reading

- [OpenSSH Official Documentation](https://www.openssh.com/manual.html)
- [PuTTY Documentation](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)
- [VS Code Remote Development](https://code.visualstudio.com/docs/remote/ssh)
- [Emacs TRAMP Manual](https://www.gnu.org/software/emacs/manual/html_node/tramp/index.html)
- [SFTP Public Key Authentication Guide — JScape](https://www.jscape.com/blog/setting-up-sftp-public-key-authentication-command-line)
- **Reference Video:** [Text Editors: Key Authentication (YouTube)](https://youtu.be/8Mt7SH2Voi0?si=Xq18AweCtW8-XGll)

---

> **Contributing:** If you find errors or have suggestions for additional editors/tools, feel free to open an issue or submit a pull request.
>
> **License:** This guide is provided as-is for educational purposes. Use at your own risk.

---

*Last updated: August 2026*
