# Deploying a Simple Python Web Server

> A step-by-step guide to building, running, and deploying a Python web application from local development to production.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Part 1: Running a Local Python Web Server](#part-1-running-a-local-python-web-server)
   - [Using Python's Built-in HTTP Server](#using-pythons-built-in-http-server)
   - [Building a Simple Flask Application](#building-a-simple-flask-application)
4. [Part 2: Preparing for Production](#part-2-preparing-for-production)
   - [Why Not Use the Development Server in Production?](#why-not-use-the-development-server-in-production)
   - [Project Structure](#project-structure)
   - [Dependencies & Virtual Environments](#dependencies--virtual-environments)
5. [Part 3: Production Application Server (Gunicorn)](#part-3-production-application-server-gunicorn)
   - [What is WSGI?](#what-is-wsgi)
   - [Installing & Running Gunicorn](#installing--running-gunicorn)
   - [Configuring Gunicorn Workers](#configuring-gunicorn-workers)
6. [Part 4: Reverse Proxy with Nginx](#part-4-reverse-proxy-with-nginx)
   - [What is a Reverse Proxy?](#what-is-a-reverse-proxy)
   - [Installing Nginx](#installing-nginx)
   - [Configuring Nginx](#configuring-nginx)
7. [Part 5: Process Management with systemd](#part-5-process-management-with-systemd)
   - [Creating a systemd Service](#creating-a-systemd-service)
   - [Managing the Service](#managing-the-service)
8. [Part 6: HTTPS with Let's Encrypt](#part-6-https-with-lets-encrypt)
   - [Installing Certbot](#installing-certbot)
   - [Obtaining & Auto-Renewing Certificates](#obtaining--auto-renewing-certificates)
9. [Part 7: Domain & DNS Setup](#part-7-domain--dns-setup)
10. [Part 8: Security Hardening](#part-8-security-hardening)
11. [Complete Deployment Checklist](#complete-deployment-checklist)
12. [Troubleshooting](#troubleshooting)
13. [References](#references)

---

## Overview

This guide walks you through the complete process of deploying a Python web application to a production environment. We'll cover:

- **Local Development**: Testing your app on your own machine
- **Production Stack**: Gunicorn (WSGI server) + Nginx (reverse proxy)
- **Process Management**: Keeping your app running with systemd
- **HTTPS**: Securing traffic with free SSL/TLS certificates
- **Security**: Best practices for a hardened deployment

> **Target Audience**: Beginners to intermediate developers who want to understand how Python web apps are deployed on a VPS (Virtual Private Server).

---

## Prerequisites

Before starting, ensure you have:

- **Python 3.8+** installed on your local machine and server
- **A VPS or cloud server** (e.g., DigitalOcean, AWS EC2, Linode, Hetzner)
- **A domain name** (optional but recommended for HTTPS)
- **Basic command line knowledge**
- **SSH access** to your server

---

## Part 1: Running a Local Python Web Server

### Using Python's Built-in HTTP Server

Python includes a simple HTTP file server module perfect for quickly sharing static files or previewing HTML.

```bash
# Serve the current directory on port 8000
python3 -m http.server 8000
```

Now open `http://localhost:8000` in your browser.

**What it does:**
- Serves static files (HTML, CSS, JS, images) from the current directory
- Requires **zero installation**
- Great for quick previews, **not for production**

**Limitations:**
- No routing, templating, or database support
- Single-threaded (handles one request at a time)
- Not secure for public exposure

---

### Building a Simple Flask Application

For dynamic web applications, we need a web framework. **Flask** is a lightweight, beginner-friendly option.

#### Step 1: Install Flask

```bash
# Create a project directory
mkdir myapp && cd myapp

# Create a virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate  # Windows

# Install Flask
pip install Flask
```

#### Step 2: Create the Application

Create a file named `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return '<h1>Hello, World!</h1><p>My first deployed Python web app.</p>'

@app.route('/about')
def about():
    return '<h1>About</h1><p>This is a simple Flask application.</p>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

#### Step 3: Run Locally

```bash
python app.py
```

Visit `http://localhost:5000` to see your app running.

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| **Route** | A URL path (`/`, `/about`) mapped to a Python function |
| **View Function** | The Python function that handles a request and returns a response |
| **Debug Mode** | Auto-reloads on code changes and shows detailed error pages |
| **host='0.0.0.0'** | Makes the server accessible from other devices on the network |

---

## Part 2: Preparing for Production

### Why Not Use the Development Server in Production?

Flask's built-in server (`app.run()`) is **not suitable for production** because it:

- Is **single-threaded** — can only handle one request at a time
- Is **not hardened** against security vulnerabilities
- Exposes **debug information** that attackers can exploit
- Has **no built-in process management** — crashes stop the app

> **Rule of thumb**: Never run `app.run()` or `python -m http.server` in production. Always use a production WSGI server.

---

### Project Structure

Organize your project for clarity and maintainability:

```
myapp/
├── app.py              # Main application entry point
├── requirements.txt    # Python dependencies
├── .env                # Environment variables (not in git!)
├── static/             # CSS, JS, images
│   ├── css/
│   └── js/
├── templates/          # HTML templates
│   ├── base.html
│   └── index.html
└── wsgi.py             # WSGI entry point for production
```

#### `wsgi.py` — Production Entry Point

```python
from app import app

if __name__ == '__main__':
    app.run()
```

This file is what Gunicorn will use to start your application.

---

### Dependencies & Virtual Environments

#### Create `requirements.txt`

```bash
pip freeze > requirements.txt
```

Example content:

```text
Flask==3.0.0
gunicorn==21.2.0
```

> **Best Practice**: Pin your dependencies to specific versions to ensure reproducible builds.

#### Environment Variables

Store configuration in environment variables (12-Factor App methodology):

```bash
# .env
FLASK_ENV=production
SECRET_KEY=your-secret-key-here
```

Load them in your app:

```python
import os
from flask import Flask

app = Flask(__name__)
app.secret_key = os.environ.get('SECRET_KEY', 'fallback-key')
```

---

## Part 3: Production Application Server (Gunicorn)

### What is WSGI?

**WSGI** (Web Server Gateway Interface) is the standard interface between Python web applications and web servers. It defines how a web server communicates with a Python app.

Think of it as a **contract**:
- Your Flask app implements the WSGI interface
- Gunicorn (a WSGI server) knows how to speak WSGI
- Together, they can handle production traffic

**Why Gunicorn?**
- **Pre-fork worker model**: Creates multiple worker processes to handle concurrent requests
- **Production-hardened**: Built for reliability and performance
- **Easy to configure**: Simple command-line options and config files

---

### Installing & Running Gunicorn

```bash
# Install Gunicorn
pip install gunicorn

# Run with 4 worker processes
gunicorn --workers 4 --bind 0.0.0.0:8000 wsgi:app
```

**Breakdown:**

| Flag | Meaning |
|------|---------|
| `--workers 4` | Run 4 worker processes |
| `--bind 0.0.0.0:8000` | Listen on all interfaces, port 8000 |
| `wsgi:app` | Import `app` from `wsgi.py` |

Now your app is accessible at `http://your-server-ip:8000`.

---

### Configuring Gunicorn Workers

The number of workers affects performance and resource usage:

```bash
# Rule of thumb: (2 x CPU cores) + 1
gunicorn --workers 9 --bind 0.0.0.0:8000 wsgi:app
```

**Worker Types:**

| Type | Best For | Command |
|------|----------|---------|
| **sync** (default) | CPU-bound, typical web apps | `--worker-class sync` |
| **gevent** | I/O-bound, many concurrent connections | `--worker-class gevent` |

**Configuration File (`gunicorn.conf.py`):**

```python
# gunicorn.conf.py
bind = "0.0.0.0:8000"
workers = 4
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2
errorlog = "/var/log/gunicorn/error.log"
accesslog = "/var/log/gunicorn/access.log"
capture_output = True
enable_stdio_inheritance = True
```

Run with:
```bash
gunicorn -c gunicorn.conf.py wsgi:app
```

---

## Part 4: Reverse Proxy with Nginx

### What is a Reverse Proxy?

A **reverse proxy** sits between clients (browsers) and your application server (Gunicorn). It:

- **Receives client requests** on ports 80 (HTTP) and 443 (HTTPS)
- **Forwards them** to Gunicorn running locally (e.g., port 8000)
- **Returns responses** back to the client

**Why Nginx?**
- **Static file serving**: Much faster than Python for CSS/JS/images
- **SSL/TLS termination**: Handles HTTPS encryption/decryption
- **Load balancing**: Distributes traffic across multiple app instances
- **Security buffer**: Shields Gunicorn from direct internet exposure
- **Compression**: Gzip responses to reduce bandwidth

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │─────▶│    Nginx    │─────▶│  Gunicorn   │
│  (Browser)  │◀─────│  (Port 80)  │◀─────│  (Port 8000)│
└─────────────┘      └─────────────┘      └─────────────┘
```

---

### Installing Nginx

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

Verify: Visit `http://your-server-ip` — you should see the Nginx welcome page.

---

### Configuring Nginx

Create a new site configuration:

```bash
sudo nano /etc/nginx/sites-available/myapp
```

Add the following:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Logging
    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;

    # Static files (served directly by Nginx)
    location /static/ {
        alias /var/www/myapp/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Proxy all other requests to Gunicorn
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}
```

**Enable the site:**

```bash
# Create symlink to sites-enabled
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

**Key Directives Explained:**

| Directive | Purpose |
|-----------|---------|
| `proxy_pass` | Forwards requests to Gunicorn |
| `proxy_set_header` | Preserves original client info (IP, protocol) |
| `location /static/` | Serves static files directly (faster) |
| `expires 30d` | Caches static files in browsers for 30 days |

---

## Part 5: Process Management with systemd

### Why systemd?

You need your app to:
- **Start automatically** on server boot
- **Restart** if it crashes
- **Run in the background** without an active terminal
- **Log output** for debugging

**systemd** is Linux's standard service manager that handles all of this.

---

### Creating a systemd Service

Create a service file:

```bash
sudo nano /etc/systemd/system/myapp.service
```

Add the following:

```ini
[Unit]
Description=Gunicorn instance to serve myapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myapp
Environment="PATH=/var/www/myapp/venv/bin"
Environment="SECRET_KEY=your-production-secret-key"
Environment="FLASK_ENV=production"
ExecStart=/var/www/myapp/venv/bin/gunicorn --workers 4 --bind 127.0.0.1:8000 wsgi:app

# Restart policy
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Breakdown:**

| Section | Purpose |
|---------|---------|
| `[Unit]` | Metadata and dependencies |
| `[Service]` | How to run the app |
| `[Install]` | When to start the service |
| `User=www-data` | Run as a non-root user for security |
| `Restart=on-failure` | Auto-restart if the app crashes |

---

### Managing the Service

```bash
# Reload systemd to recognize the new service
sudo systemctl daemon-reload

# Start the service
sudo systemctl start myapp

# Enable auto-start on boot
sudo systemctl enable myapp

# Check status
sudo systemctl status myapp

# View logs
sudo journalctl -u myapp -f

# Restart after code changes
sudo systemctl restart myapp
```

---

## Part 6: HTTPS with Let's Encrypt

### Why HTTPS?

- **Encrypts data** between browser and server
- **Prevents man-in-the-middle attacks**
- **Required** for modern web features (geolocation, service workers, etc.)
- **SEO boost** — Google ranks HTTPS sites higher

**Let's Encrypt** provides free SSL/TLS certificates.

---

### Installing Certbot

```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx
```

---

### Obtaining & Auto-Renewing Certificates

```bash
# Obtain certificate and auto-configure Nginx
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Follow the prompts. Certbot will:
1. Verify domain ownership
2. Obtain the certificate
3. Automatically update your Nginx config for HTTPS

**Verify auto-renewal:**

```bash
# Test renewal (dry run)
sudo certbot renew --dry-run
```

Certbot automatically sets up a cron job for renewal. Certificates renew every 90 days.

**Your Nginx config will now include:**

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ... rest of your config
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

---

## Part 7: Domain & DNS Setup

### Pointing Your Domain to Your Server

1. **Get your server's public IP address:**
   ```bash
   curl ifconfig.me
   ```

2. **Add DNS records** at your domain registrar:

   | Type | Name | Value | TTL |
   |------|------|-------|-----|
   | A | @ | your-server-ip | 3600 |
   | A | www | your-server-ip | 3600 |

3. **Wait for propagation** (usually minutes to a few hours).

4. **Verify:**
   ```bash
   dig your-domain.com +short
   ```

---

## Part 8: Security Hardening

### Essential Security Steps

#### 1. Firewall (UFW)

```bash
# Allow SSH, HTTP, and HTTPS only
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable

# Check status
sudo ufw status
```

#### 2. Secure SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Set:
```text
PermitRootLogin no
PasswordAuthentication no  # Use SSH keys only
Port 2222                  # Change default port (optional)
```

```bash
sudo systemctl restart sshd
```

#### 3. Keep Software Updated

```bash
sudo apt update && sudo apt upgrade -y
```

#### 4. Run App as Non-Root User

Your systemd service already runs as `www-data`. Never run web apps as root.

#### 5. Environment Variables for Secrets

Never hardcode secrets in your code. Always use environment variables.

#### 6. Security Headers in Nginx

Add to your Nginx config:

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

---

## Complete Deployment Checklist

Use this checklist before going live:

- [ ] Code tested locally and committed to version control
- [ ] Dependencies pinned in `requirements.txt`
- [ ] Secrets stored in environment variables (not in code)
- [ ] Production WSGI server (Gunicorn) configured
- [ ] Nginx reverse proxy configured with static file serving
- [ ] Domain DNS records pointing to server IP
- [ ] HTTPS enabled with Let's Encrypt
- [ ] systemd service created and enabled
- [ ] Firewall configured (UFW)
- [ ] SSH secured (keys only, root login disabled)
- [ ] Logging configured and accessible
- [ ] Health check endpoint available (`/health`)

---

## Troubleshooting

### Common Issues

#### 502 Bad Gateway

**Cause**: Nginx can't connect to Gunicorn.

**Solutions:**
```bash
# Check if Gunicorn is running
sudo systemctl status myapp

# Check Gunicorn logs
sudo journalctl -u myapp

# Verify binding address matches Nginx proxy_pass
# Gunicorn should bind to 127.0.0.1:8000 (not 0.0.0.0)
```

#### Permission Denied Errors

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/myapp

# Fix permissions
sudo chmod -R 755 /var/www/myapp
```

#### Static Files Not Loading

- Ensure `location /static/` path in Nginx matches your actual static files directory
- Run `sudo nginx -t` to check for config syntax errors
- Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`

#### Certificate Issues

```bash
# Check certificate status
sudo certbot certificates

# Force renewal
sudo certbot renew --force-renewal
```

---

## References

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Gunicorn Documentation](https://docs.gunicorn.org/)
- [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [The Twelve-Factor App](https://12factor.net/)
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)

---

## License

This guide is provided as-is for educational purposes. Feel free to adapt it for your own projects.

> **Video Reference**: [Deploying a Simple Python Web Server](https://youtu.be/KgAtZ1LlNiQ)
