# PART 5 — GitHub Actions (Automated CI/CD)

## Why Use GitHub Actions?

Instead of manually doing this every time you update code:

1. Build on Mac
2. Copy to server manually
3. SSH in to restart PM2

GitHub Actions automates all of this after every `git push`, builds on GitHub's servers (free), and uses no EC2 RAM.

## Step 5.1 — Create a Dedicated SSH Key for GitHub Actions

**Why:** GitHub Actions needs to SSH into EC2 to copy files and restart PM2. Create a separate key for access control — don't share your personal key.

```bash
# On SERVER
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github-actions

# Add public key to authorized_keys
cat ~/.ssh/github-actions.pub >> ~/.ssh/authorized_keys

# Print private key to add to GitHub Secrets
cat ~/.ssh/github-actions
```

## Step 5.2 — Add Secrets to GitHub

**Why:** Never hardcode your IP, username, or SSH key in the workflow file — it exposes sensitive information. GitHub Secrets encrypts and injects them at runtime.

GitHub repo → Settings → Secrets and variables → Actions → New repository secret:

| Name | Value |
|------|-------|
| `EC2_HOST` | Your Elastic IP |
| `EC2_USER` | `ubuntu` |
| `EC2_SSH_KEY` | Private key content (`cat ~/.ssh/github-actions`) |

## Step 5.3 — Create Workflow File

**Why:** The `.yml` file defines the steps GitHub Actions will run automatically on every push to the specified branch.

```bash
# On Mac
mkdir -p .github/workflows
nano .github/workflows/deploy.yml
```

```yaml
name: Deploy BE

on:
  push:
    branches:
      - main  # change to your branch name

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install & Build
        run: |
          npm install
          npm run build

      - name: Copy dist to EC2
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "dist/"
          target: "~/productivity-plogg-backend/"

      - name: Restart PM2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            pm2 restart my-back-end
```

## Step 5.4 — Push to GitHub

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deploy"
git push
```
