# PART 3 — Deploy Code (NestJS Backend)

## Step 3.1 — Prepare Directory on Server

**Why:** The directory must exist before copying files, otherwise `scp` throws an error.

```bash
# On server
mkdir -p ~/backend
```

## Step 3.2 — Create .env File on Server

**Why:** Environment variables contain sensitive data (DB credentials, JWT secrets, etc.) that should never be committed to GitHub. Create it manually on the server.

```bash
nano ~/backend/.env
# Paste your .env contents
# Ctrl+X → Y → Enter to save
```

## Step 3.3 — Copy package.json to Server

**Why:** The server needs `package.json` to know which packages to install.

```bash
# On Mac
scp -i ~/.ssh/my-back-end-key-pair.pem ./package.json ubuntu@YOUR_IP:~/backend/
```

## Step 3.4 — Build on Mac, Copy to Server

**Why:** t2.micro only has 1GB RAM, which is not enough to compile TypeScript (NestJS build needs ~500MB RAM). Build on Mac and copy the compiled files to the server.

> ⚠️ If using Supabase Realtime with Node.js 20, use `import * as ws from 'ws'` instead of `import ws from 'ws'` to avoid WebSocket errors.

```bash
# On Mac — build
npm run build

# Compress dist/ for faster transfer
zip -r dist.zip dist/

# Copy to server
scp -i ~/.ssh/my-back-end-key-pair.pem dist.zip ubuntu@YOUR_IP:~/backend/
```

> 💡 Use `rsync` to only copy changed files next time (much faster):

```bash
rsync -avz --progress -e "ssh -i ~/.ssh/my-back-end-key-pair.pem" ./dist ubuntu@YOUR_IP:~/backend/
```

## Step 3.5 — Extract on Server

```bash
# On server
cd ~/backend
rm -rf dist/      # remove old dist if exists
unzip dist.zip
rm dist.zip       # clean up zip file
```

## Step 3.6 — Install node_modules on Server

**Why:** node_modules must be installed on the Linux server because some packages have different native binaries between Mac and Linux. Never copy node_modules from Mac to the server.

```bash
# On server
cd ~/backend
npm install
```

> ⚠️ Some packages like `class-transformer` and `class-validator` are easily put into `devDependencies` by mistake. If the app reports `Cannot find module`, check and move them to `dependencies` in `package.json` on Mac, then `npm install` again on the server.

## Step 3.7 — Run App with PM2

**Why:** `dist/main.js` is the entry point after compiling TypeScript → JavaScript. PM2 keeps the process running continuously.

```bash
pm2 start ~/backend/dist/main.js --name "backend"
pm2 startup   # auto-start on reboot
pm2 save      # save process list
```

## Step 3.8 — Verify App is Running

```bash
# Check which port is listening
ss -tlnp | grep node

# Test the API
curl http://localhost:8000
```
