#  Environment Variables — Complete Guide
---


## 1. What Are Environment Variables?

An **environment variable** is a dynamically set, user-definable **key-value pair** that affects how running processes behave on a computer. They are part of the environment in which a process runs.

>  **Think of it as:** A configuration setting that lives *outside* your code but can be read *by* your code at runtime.

### Basic Structure

```
KEY=value
```

| Component | Description | Example |
|-----------|-------------|---------|
| `KEY` | The name of the variable (conventionally UPPER_CASE) | `DATABASE_URL` |
| `=` | Assignment operator (no spaces around it!) | `=` |
| `value` | The data assigned to the key | `postgres://localhost:5432/mydb` |

### Example in Code

**❌ Hardcoded (Bad):**
```javascript
const apiKey = "sk-live-1234567890abcdef";
```

**✅ Using Environment Variable (Good):**
```javascript
const apiKey = process.env.API_KEY;
```

At runtime, `process.env.API_KEY` is replaced with the actual value stored in the environment.

---

## 2. Why Use Environment Variables?

### 🔒 1. Security
Keep secrets (API keys, passwords, tokens) out of your source code and version control.

> **Problem:** If you hardcode a secret API key and push to GitHub, anyone can see and abuse it.
>
> **Solution:** Store the key in an environment variable and reference it in code.

### 🔄 2. Configuration Without Code Changes
Change behavior across environments (dev, staging, production) without modifying or redeploying code.

| Environment | `NODE_ENV` | `DATABASE_URL` |
|-------------|------------|----------------|
| Development | `development` | `localhost:5432/dev_db` |
| Staging | `staging` | `staging-db.example.com:5432` |
| Production | `production` | `prod-db.example.com:5432` |

### 🧪 3. Portability
The same codebase can run on different machines, servers, or containers with different configurations.

### 🤝 4. Team Collaboration
Each developer can have their own local settings without conflicting with others.

---

## 3. Types of Environment Variables

### By Scope

| Type | Scope | Persistence | Example |
|------|-------|-------------|---------|
| **System** | All users & processes | Reboot persists | `PATH`, `TEMP` |
| **User** | Current user only | Reboot persists | `HOME`, `USERPROFILE` |
| **Session** | Current terminal/session | Lost on close | `NODE_ENV=development` |
| **Process** | Specific running app | Lost when app exits | App-specific configs |

### By Inheritance

- **Parent → Child:** By default, child processes inherit all environment variables from their parent.
- **Isolation:** Changes in a child process do NOT affect the parent or sibling processes.

```
Terminal (Parent)
  ├── Shell Script (Child) → inherits all env vars
  │     └── Node.js App (Grandchild) → inherits all env vars
  └── Another Script (Sibling) → unaffected by changes in the first child
```

---

## 4. Common Use Cases

| Use Case | Example Variable | Purpose |
|----------|------------------|---------|
| **API Keys** | `OPENAI_API_KEY` | Authenticate with external services |
| **Database URLs** | `DATABASE_URL` | Connect to different databases per environment |
| **Execution Mode** | `NODE_ENV` | Toggle between `development`, `production`, `test` |
| **Port Numbers** | `PORT` | Run app on different ports |
| **Feature Flags** | `ENABLE_BETA_FEATURES` | Toggle features without deploying |
| **Log Levels** | `LOG_LEVEL` | Control verbosity (`debug`, `info`, `warn`, `error`) |
| **Email Config** | `SMTP_HOST`, `SMTP_PORT` | Configure email services |
| **Domain Names** | `API_BASE_URL` | Point to different API endpoints |

---

## 5. Setting Environment Variables

### Linux / macOS

#### Temporary (Session-only)
```bash
# Method 1: Inline (only for this command)
NODE_ENV=production node app.js

# Method 2: export (persists for current session)
export API_KEY="sk-live-12345"
export PORT=3000

# Verify
echo $API_KEY
printenv API_KEY
```

#### Permanent (User-level)
Add to your shell profile file:

```bash
# For Bash (~/.bashrc)
echo 'export API_KEY="sk-live-12345"' >> ~/.bashrc
source ~/.bashrc

# For Zsh (~/.zshrc)
echo 'export API_KEY="sk-live-12345"' >> ~/.zshrc
source ~/.zshrc
```

#### View All Environment Variables
```bash
printenv        # Show all
printenv PATH   # Show specific variable
env             # Alternative
```

#### Unset a Variable
```bash
unset API_KEY
```

---

### Windows

#### Command Prompt (cmd.exe)
```cmd
:: Set for current session
set API_KEY=sk-live-12345

:: Set permanently (requires restart)
setx API_KEY "sk-live-12345"

:: View all
set

:: View specific
set API_KEY

:: Unset
set API_KEY=
```

#### PowerShell
```powershell
# Set for current session
$env:API_KEY = "sk-live-12345"

# Set permanently (User scope)
[Environment]::SetEnvironmentVariable("API_KEY", "sk-live-12345", "User")

# Set permanently (Machine scope — requires admin)
[Environment]::SetEnvironmentVariable("API_KEY", "sk-live-12345", "Machine")

# View
$env:API_KEY
Get-ChildItem Env:

# Unset
Remove-Item Env:API_KEY
```

#### GUI Method
1. Press `Win + R`, type `sysdm.cpl`, press Enter
2. Go to **Advanced** → **Environment Variables**
3. Add under **User variables** or **System variables**

---

### Node.js / JavaScript

#### Reading Environment Variables
```javascript
// Access any environment variable
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 3000;  // Fallback default

// Check if in production
if (process.env.NODE_ENV === 'production') {
  console.log('Running in production mode');
}
```

#### Setting in package.json Scripts
```json
{
  "scripts": {
    "start": "NODE_ENV=production node app.js",
    "dev": "NODE_ENV=development nodemon app.js",
    "test": "NODE_ENV=test jest"
  }
}
```

> ⚠️ **Windows users:** Use `cross-env` package for cross-platform compatibility:
> ```bash
> npm install cross-env
> ```
> ```json
> {
>   "scripts": {
>     "start": "cross-env NODE_ENV=production node app.js"
>   }
> }
> ```

---

### Python

```python
import os

# Read
api_key = os.environ.get('API_KEY')
# OR with default
port = os.environ.get('PORT', '5000')

# Check existence
if 'DEBUG' in os.environ:
    print("Debug mode is on")

# Set (current process only)
os.environ['NEW_VAR'] = 'value'

# Using python-dotenv library
from dotenv import load_dotenv
load_dotenv()  # Loads .env file
```

---

## 6. The `.env` File

A `.env` file is a local configuration file that stores environment variables for development. It is **NOT** committed to version control.

### Structure

```env
# .env — Local Development Configuration
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgres://localhost:5432/myapp_dev

# API Keys
OPENAI_API_KEY=sk-dev-1234567890
STRIPE_SECRET_KEY=sk_test_xxxxxxxx

# Email
SMTP_HOST=smtp.mailtrap.io
SMTP_PORT=2525

# Feature Flags
ENABLE_BETA_FEATURES=true
```

### Rules

| Rule | Why |
|------|-----|
| **No spaces around `=`** | `KEY=value` ✅ — `KEY = value` ❌ |
| **Quote values with spaces** | `MESSAGE="Hello World"` |
| **No quotes needed for simple values** | `PORT=3000` |
| **Comments start with `#`** | `# This is a comment` |
| **One variable per line** | No multi-line values (unless quoted) |

### Loading `.env` in Your App

#### Node.js (using `dotenv`)
```bash
npm install dotenv
```

```javascript
// At the VERY TOP of your entry file (e.g., index.js)
require('dotenv').config();

// Now you can use process.env anywhere
console.log(process.env.DATABASE_URL);
```

#### Python (using `python-dotenv`)
```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
load_dotenv()
```

### ⚠️ CRITICAL: `.gitignore`

**NEVER** commit `.env` files to Git!

```gitignore
# .gitignore
.env
.env.local
.env.*.local
```

### `.env.example` (Template)

Commit a template file so teammates know what variables they need:

```env
# .env.example — Copy this to .env and fill in your values
NODE_ENV=development
PORT=3000
DATABASE_URL=
OPENAI_API_KEY=
STRIPE_SECRET_KEY=
SMTP_HOST=
SMTP_PORT=
```

---

## 7. Best Practices

### ✅ DO

| Practice | Reason |
|----------|--------|
| **Use UPPER_CASE names** | Convention — easy to spot (`API_KEY`) |
| **Use descriptive names** | `DATABASE_URL` > `DB_U` |
| **Provide defaults** | `const port = process.env.PORT \|\| 3000;` |
| **Use `.env` for local dev** | Keeps local config isolated |
| **Document required variables** | README or `.env.example` |
| **Validate at startup** | Fail fast if required vars are missing |
| **Use different files per environment** | `.env.development`, `.env.production` |

### ❌ DON'T

| Practice | Reason |
|----------|--------|
| **Hardcode secrets** | Security risk if code is exposed |
| **Commit `.env` to Git** | Secrets will be exposed forever |
| **Log sensitive values** | Never `console.log(process.env.SECRET_KEY)` |
| **Use environment variables for rapidly changing data** | They are meant for config, not state |
| **Rely on environment variables in frontend code** | They get bundled into client-side JS |

### Validation Example (Node.js)

```javascript
const requiredEnvVars = ['DATABASE_URL', 'JWT_SECRET', 'API_KEY'];

requiredEnvVars.forEach((varName) => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```

---

## 8. Common Environment Variables

### System-Level (Built-in)

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `PATH` | List of directories to search for executables | `/usr/local/bin:/usr/bin` |
| `HOME` | Current user's home directory | `/home/username` |
| `USER` / `USERNAME` | Current username | `john_doe` |
| `SHELL` | Default shell path | `/bin/bash` |
| `PWD` | Present working directory | `/home/user/projects` |
| `TEMP` / `TMP` | Temporary file directory | `/tmp` |
| `LANG` | System language/locale | `en_US.UTF-8` |

### Development (Common)

| Variable | Typical Use |
|----------|-------------|
| `NODE_ENV` | `development`, `production`, `test` |
| `PORT` | Server port number |
| `DATABASE_URL` | Full database connection string |
| `JWT_SECRET` | Secret key for JWT token signing |
| `API_KEY` | External service authentication |
| `LOG_LEVEL` | `debug`, `info`, `warn`, `error` |
| `DEBUG` | Enable debug output (e.g., `app:*`) |

### Frontend Framework Prefixes

| Prefix | Framework | Visibility |
|--------|-----------|------------|
| `REACT_APP_` | Create React App | Public (bundled) |
| `NEXT_PUBLIC_` | Next.js | Public (bundled) |
| `VITE_` | Vite | Public (bundled) |
| `NUXT_PUBLIC_` | Nuxt.js | Public (bundled) |
| `GATSBY_` | Gatsby | Public (bundled) |
| `PUBLIC_` | SvelteKit | Public (bundled) |

> ⚠️ **Important:** Only prefix variables with these if they are **safe to expose to the browser**. Never prefix secrets!

---

## 9. Security Considerations

### 🔴 High Risk

1. **Never commit secrets**
   ```bash
   # Check what you're about to commit
   git diff --cached
   ```

2. **Rotate exposed secrets immediately**
   If a secret is accidentally committed, treat it as compromised.

3. **Use secret managers in production**
   - AWS Secrets Manager
   - Azure Key Vault
   - HashiCorp Vault
   - Google Secret Manager
   - Doppler

4. **Restrict frontend exposure**
   ```javascript
   // ❌ DON'T — This gets bundled into client JS!
   const secret = process.env.SECRET_KEY;

   // ✅ DO — Only use public-prefixed vars in frontend
   const publicKey = process.env.REACT_APP_PUBLIC_KEY;
   ```

5. **Validate and sanitize**
   Don't trust environment variable values blindly.

### 🟡 Medium Risk

- Use `.env.local` for machine-specific overrides
- Keep `.env` files out of Docker images:
  ```dockerfile
  # Dockerfile
  .env
  ```

---

## 10. Quick Reference Cheat Sheet

### Linux / macOS Commands

| Task | Command |
|------|---------|
| Set (session) | `export VAR=value` |
| Set (inline) | `VAR=value command` |
| View all | `printenv` or `env` |
| View one | `echo $VAR` or `printenv VAR` |
| Unset | `unset VAR` |
| Add to PATH | `export PATH=$PATH:/new/path` |

### Windows Commands

| Task | CMD | PowerShell |
|------|-----|------------|
| Set (session) | `set VAR=value` | `$env:VAR = "value"` |
| Set (permanent) | `setx VAR "value"` | `[Environment]::SetEnvironmentVariable("VAR", "value", "User")` |
| View all | `set` | `Get-ChildItem Env:` |
| View one | `set VAR` | `$env:VAR` |
| Unset | `set VAR=` | `Remove-Item Env:VAR` |

### Node.js

| Task | Code |
|------|------|
| Read | `process.env.VAR_NAME` |
| Read with default | `process.env.VAR_NAME \|\| 'default'` |
| Check existence | `'VAR_NAME' in process.env` |

### `.env` File Template

```env
# Application
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=your_database_url_here

# External Services
API_KEY=your_api_key_here
SECRET_KEY=your_secret_here

# Optional
LOG_LEVEL=debug
```

---

## 📚 Further Reading

- [Node.js `process.env` Documentation](https://nodejs.org/api/process.html#process_process_env)
- [The Twelve-Factor App: Config](https://12factor.net/config)
- [dotenv npm package](https://www.npmjs.com/package/dotenv)
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [OWASP: Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

---

## 📝 License

This guide is provided as educational material. Feel free to use, modify, and share.

---

> 💬 **Questions or suggestions?** Open an issue or submit a PR!
