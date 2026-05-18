# PART 6 — Overall Workflow Overview

```
Mac (local)
    ↓ git push
GitHub
    ↓ trigger GitHub Actions
GitHub Servers (free build)
    ↓ npm install + npm run build
    ↓ copy dist/ to EC2
    ↓ pm2 restart
EC2 Server (Ubuntu)
    ↓
  Nginx (port 80)
    ↓
NestJS App (port 8000, PM2)
    ↓
  Database
    ↑
  User
```
