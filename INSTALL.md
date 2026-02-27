# One-Line Installation Guide

Quick deployment of Daily Digest API using the automated installer.

## 🚀 Quick Install

### On Ubuntu/Debian Server:

```bash
curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/daily-digest/main/install.sh | bash
```

That's it! The script will:
- ✅ Install all dependencies
- ✅ Set up Python environment
- ✅ Clone the application
- ✅ Ask for your API keys interactively
- ✅ Create systemd service
- ✅ Start the application

## 📋 What the Installer Does

### 1. System Check
- Detects OS (Ubuntu/Debian)
- Checks compatibility

### 2. Install Dependencies
- Python 3
- pip & venv
- git, curl, etc.

### 3. Download Application
- Clones from GitHub OR
- Downloads release archive

### 4. Setup Python Environment
- Creates virtual environment
- Installs all Python packages

### 5. Interactive Configuration
Asks you for each API key:
- Claude AI (Anthropic)
- Gmail API credentials
- NewsAPI key
- Weather API key
- Todoist token
- Default location
- etc.

Press Enter to skip any API key!

### 6. Systemd Service (Optional)
- Creates auto-start service
- Enables on boot
- Configures automatic restarts

### 7. Start Application
- Starts the API
- Verifies it's running

## 🎯 Installation Walkthrough

```bash
# Run the installer
curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/daily-digest/main/install.sh | bash

# You'll see:
╔════════════════════════════════════════════════════════════╗
║           Daily Digest API - Installation Script          ║
╚════════════════════════════════════════════════════════════╝

❓ Continue with installation? (Y/n): y

✓ Detected: Ubuntu 22.04
✓ Installing system dependencies...
✓ Setting up application...
✓ Python environment configured

# Then it asks for API keys:
❓ Claude API Key (sk-ant-...): sk-ant-xxxxx
✓ Claude AI configured

❓ Configure Gmail API? (y/N): n
❓ Configure NewsAPI? (y/N): y
❓ NewsAPI Key: xxxxx
✓ NewsAPI configured

❓ Default location (for weather/traffic) [New York, NY]: San Francisco

❓ Set up systemd service (auto-start on boot)? (Y/n): y
✓ Systemd service created and enabled

❓ Start the application now? (Y/n): y
✓ Application started via systemd

Installation Complete! 🎉
```

## 🔧 Post-Installation

### Check if it's running:
```bash
curl http://localhost:8000/health
```

### Access the API:
```bash
# Health check
curl http://localhost:8000/health

# Generate digest
curl -X POST http://localhost:8000/digest/quick

# API documentation
open http://localhost:8000/docs
```

### Manage the service:
```bash
# Check status
sudo systemctl status daily-digest

# View logs
sudo journalctl -u daily-digest -f

# Restart
sudo systemctl restart daily-digest

# Stop
sudo systemctl stop daily-digest
```

### Add missing API keys:
```bash
cd ~/daily-digest
nano .env
# Edit the file, save, then:
sudo systemctl restart daily-digest
```

## 🌐 Making It Publicly Accessible

### Setup nginx reverse proxy:

```bash
sudo apt install nginx

# Create config
sudo nano /etc/nginx/sites-available/daily-digest
```

Add this configuration:
```nginx
server {
    listen 80;
    server_name your-domain.com;  # or your IP

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/daily-digest /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Open firewall
sudo ufw allow 80
sudo ufw allow 443  # For HTTPS
```

### Add HTTPS (recommended):
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

## 📦 How to Host the Install Script

### Option 1: GitHub (Recommended)

1. **Push your code to GitHub**:
   ```bash
   cd ~/Documents/Tutorials/12_Daily_DIgest
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/daily-digest.git
   git push -u origin main
   ```

2. **Update install.sh**:
   Open `install.sh` and change:
   ```bash
   GITHUB_REPO="YOUR_USERNAME/daily-digest"
   ```

3. **Your install command becomes**:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/daily-digest/main/install.sh | bash
   ```

### Option 2: Your Own Server

1. **Host the script**:
   ```bash
   # On your web server
   sudo cp install.sh /var/www/html/install.sh
   ```

2. **Install command**:
   ```bash
   curl -fsSL https://yourdomain.com/install.sh | bash
   ```

### Option 3: GitHub Gist

1. Create a gist at https://gist.github.com/
2. Paste `install.sh` content
3. Get the raw URL
4. Use:
   ```bash
   curl -fsSL https://gist.githubusercontent.com/YOUR_USERNAME/GIST_ID/raw/install.sh | bash
   ```

## 🔒 Security Considerations

### Piping to bash is convenient but potentially risky!

**Safe practices**:

1. **Review before running**:
   ```bash
   # Download and inspect first
   curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/daily-digest/main/install.sh > install.sh
   less install.sh  # Review the script
   bash install.sh  # Run if satisfied
   ```

2. **Use HTTPS only** (not HTTP)

3. **Pin to a specific version**:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/daily-digest/v1.0.0/install.sh | bash
   ```

4. **Verify checksum** (advanced):
   ```bash
   curl -fsSL https://example.com/install.sh | tee install.sh | sha256sum
   # Compare with published checksum
   bash install.sh
   ```

## 🐛 Troubleshooting

### "Permission denied" errors:
```bash
# Run with sudo if needed (not recommended for whole script)
# Better: fix permissions after
sudo chown -R $USER:$USER ~/daily-digest
```

### Python version issues:
```bash
# Ensure Python 3.8+
python3 --version

# If too old, install newer version
sudo apt install python3.11
```

### Port 8000 already in use:
```bash
# Find what's using it
sudo lsof -i :8000

# Change port in install.sh or after installation
# Edit the systemd service:
sudo nano /etc/systemd/system/daily-digest.service
# Change --port 8000 to --port 8001
sudo systemctl daemon-reload
sudo systemctl restart daily-digest
```

### Can't connect from outside:
```bash
# Check firewall
sudo ufw status
sudo ufw allow 8000

# Check if binding to 0.0.0.0
ps aux | grep uvicorn
```

## 📝 Manual Installation (Alternative)

If the auto-installer doesn't work:

```bash
# Clone repo
git clone https://github.com/YOUR_USERNAME/daily-digest.git
cd daily-digest

# Setup
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Configure
cp env.example .env
nano .env  # Add your API keys

# Run
uvicorn main:app --host 0.0.0.0 --port 8000
```

## 🎉 That's It!

Your API is now:
- ✅ Installed
- ✅ Configured
- ✅ Running
- ✅ Auto-starting on boot

Test it:
```bash
curl -X POST http://your-server-ip:8000/digest/quick
```

Set up a cron job or GitHub Action to call it daily! 🚀
