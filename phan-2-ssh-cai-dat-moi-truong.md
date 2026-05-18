# PHẦN 2 — SSH & Cài đặt môi trường

## Bước 2.1 — SSH vào server

**Lý do:** SSH (Secure Shell) cho phép điều khiển server từ xa qua terminal.

```bash
# Phân quyền file .pem (bắt buộc, nếu chưa làm)
chmod 400 ~/.ssh/my-back-end-key-pair.pem

# SSH vào server
ssh -i ~/.ssh/my-back-end-key-pair.pem ubuntu@YOUR_ELASTIC_IP
```

> ⚠️ Nếu gặp lỗi `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` (thường xảy ra khi tạo instance mới với cùng IP):

```bash
ssh-keygen -R YOUR_ELASTIC_IP
# Sau đó SSH lại và gõ "yes" khi được hỏi
```

## Bước 2.2 — Cập nhật hệ thống

**Lý do:** Ubuntu mới tạo thường có packages cũ, cần cập nhật để tránh lỗi bảo mật và conflict.

```bash
sudo apt update && sudo apt upgrade -y
```

## Bước 2.3 — Cài Node.js 20

**Lý do:** Ubuntu mặc định có Node.js cũ (v12). NodeSource cung cấp script để cài đúng version LTS mới nhất.

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v  # kiểm tra
```

## Bước 2.4 — Cài PM2

**Lý do:** Nếu chạy thẳng `node dist/main.js` thì tắt terminal là app chết. PM2 là process manager giữ app chạy liên tục 24/7, tự restart khi crash, tự khởi động lại khi server reboot.

```bash
sudo npm install -g pm2
```

## Bước 2.5 — Cài Nginx

**Lý do:** App NestJS chạy ở port 8000, nhưng người dùng truy cập qua port 80 (HTTP). Nginx đóng vai trò reverse proxy — nhận request từ user rồi chuyển vào đúng app.

```bash
sudo apt install -y nginx
sudo systemctl status nginx  # kiểm tra đang chạy chưa
```

## Bước 2.6 — Cài unzip

**Lý do:** Cần thiết để giải nén file `.zip` khi copy code từ Mac lên server.

```bash
sudo apt install -y unzip
```