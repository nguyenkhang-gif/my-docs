# PHẦN 5 — GitHub Actions (CI/CD tự động)

## Tại sao dùng GitHub Actions?

Thay vì mỗi lần update code phải:

1. Build trên Mac
2. Copy lên server thủ công
3. SSH vào restart PM2

GitHub Actions tự động làm hết sau mỗi lần `git push`, build trên máy chủ của GitHub (miễn phí), không tốn RAM của EC2.

## Bước 5.1 — Tạo SSH Key riêng cho GitHub Actions

**Lý do:** GitHub Actions cần SSH vào EC2 để copy file và restart PM2. Tạo key riêng để kiểm soát quyền truy cập, không dùng chung key cá nhân.

```bash
# Trên SERVER
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github-actions

# Thêm public key vào authorized_keys
cat ~/.ssh/github-actions.pub >> ~/.ssh/authorized_keys

# Copy private key để thêm vào GitHub Secrets
cat ~/.ssh/github-actions
```

## Bước 5.2 — Thêm Secrets vào GitHub

**Lý do:** Không được hardcode IP, username, SSH key vào file workflow vì sẽ lộ thông tin. GitHub Secrets mã hóa và inject vào lúc chạy.

GitHub repo → Settings → Secrets and variables → Actions → New repository secret:

| Name | Value |
|------|-------|
| `EC2_HOST` | Your Elastic IP |
| `EC2_USER` | `ubuntu` |
| `EC2_SSH_KEY` | Nội dung private key (`cat ~/.ssh/github-actions`) |

## Bước 5.3 — Tạo file workflow

**Lý do:** File `.yml` định nghĩa các bước GitHub Actions sẽ chạy tự động khi có push lên nhánh chỉ định.

```bash
# Trên Mac
mkdir -p .github/workflows
nano .github/workflows/deploy.yml
```

```yaml
name: Deploy BE

on:
  push:
    branches:
      - main  # đổi thành tên nhánh của bạn

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

## Bước 5.4 — Push lên GitHub

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deploy"
git push
```
