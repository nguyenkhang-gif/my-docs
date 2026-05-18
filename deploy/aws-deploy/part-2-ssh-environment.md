# PART 2 — SSH & Environment Setup

## Step 2.1 — SSH into the Server

**Why:** SSH (Secure Shell) lets you control the server remotely from your terminal.

```bash
# Set correct permissions on .pem file (required if not done yet)
chmod 400 ~/.ssh/my-back-end-key-pair.pem

# SSH into server
ssh -i ~/.ssh/my-back-end-key-pair.pem ubuntu@YOUR_ELASTIC_IP
```

> ⚠️ If you get `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` (usually happens when creating a new instance with the same IP):

```bash
ssh-keygen -R YOUR_ELASTIC_IP
# Then SSH again and type "yes" when prompted
```

## Step 2.2 — Update the System

**Why:** Freshly created Ubuntu instances often have outdated packages. Updating avoids security vulnerabilities and conflicts.

```bash
sudo apt update && sudo apt upgrade -y
```

## Step 2.3 — Install Node.js 20

**Why:** Ubuntu ships with an old Node.js (v12). NodeSource provides a script to install the latest LTS version correctly.

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v  # verify
```

## Step 2.4 — Install PM2

**Why:** Running `node dist/main.js` directly means the app dies when you close the terminal. PM2 is a process manager that keeps your app running 24/7, auto-restarts on crash, and auto-starts on server reboot.

```bash
sudo npm install -g pm2
```

## Step 2.5 — Install Nginx

**Why:** The NestJS app runs on port 8000, but users access it via port 80 (HTTP). Nginx acts as a reverse proxy — it receives requests from users and forwards them to the correct app.

```bash
sudo apt install -y nginx
sudo systemctl status nginx  # check if running
```

## Step 2.6 — Install unzip

**Why:** Needed to extract `.zip` files when copying code from your Mac to the server.

```bash
sudo apt install -y unzip
```
