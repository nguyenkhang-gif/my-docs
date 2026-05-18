# PHẦN 6 — Luồng hoạt động tổng quan

```
Máy Mac (local)
    ↓ git push
GitHub
    ↓ trigger GitHub Actions
Máy chủ GitHub (build miễn phí)
    ↓ npm install + npm run build
    ↓ copy dist/ lên EC2
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
