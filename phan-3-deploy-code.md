# PHẦN 3 — Deploy code (NestJS BE)

## Bước 3.1 — Chuẩn bị thư mục trên server

**Lý do:** Thư mục phải tồn tại trước khi copy file lên, không thì `scp` báo lỗi.

```bash
# Trên server
mkdir -p ~/backend
```

## Bước 3.2 — Tạo file .env trên server

**Lý do:** Biến môi trường chứa thông tin nhạy cảm (DB, JWT secret...) không được commit lên GitHub, phải tạo thủ công trên server.

```bash
nano ~/backend/.env
# Paste nội dung .env vào
# Ctrl+X → Y → Enter để lưu
```

## Bước 3.3 — Copy package.json lên server

**Lý do:** Server cần `package.json` để biết cài những packages nào.

```bash
# Trên Mac
scp -i ~/.ssh/my-back-end-key-pair.pem ./package.json ubuntu@YOUR_IP:~/backend/
```

## Bước 3.4 — Build trên Mac, copy lên server

**Lý do:** t2.micro chỉ có 1GB RAM, không đủ để compile TypeScript (NestJS build cần ~500MB RAM). Build trên Mac xong copy file đã compile lên server.

> ⚠️ Nếu dùng Supabase Realtime với Node.js 20, cần dùng `import * as ws from 'ws'` thay vì `import ws from 'ws'` để tránh lỗi WebSocket.

```bash
# Trên Mac — build
npm run build

# Nén dist/ để copy nhanh hơn
zip -r dist.zip dist/

# Copy lên server
scp -i ~/.ssh/my-back-end-key-pair.pem dist.zip ubuntu@YOUR_IP:~/backend/
```

> 💡 Dùng `rsync` để lần sau chỉ copy file thay đổi (nhanh hơn):

```bash
rsync -avz --progress -e "ssh -i ~/.ssh/my-back-end-key-pair.pem" ./dist ubuntu@YOUR_IP:~/backend/
```

## Bước 3.5 — Giải nén trên server

```bash
# Trên server
cd ~/backend
rm -rf dist/      # xóa dist cũ nếu có
unzip dist.zip
rm dist.zip       # dọn dẹp file zip
```

## Bước 3.6 — Cài node_modules trên server

**Lý do:** node_modules cần được cài trên Linux server vì một số packages có native binary khác nhau giữa Mac và Linux. Không copy node_modules từ Mac lên.

```bash
# Trên server
cd ~/backend
npm install
```

> ⚠️ Một số packages như `class-transformer`, `class-validator` dễ bị để nhầm vào `devDependencies`. Nếu app báo `Cannot find module`, kiểm tra và chuyển sang `dependencies` trong `package.json` trên Mac rồi `npm install` lại trên server.

## Bước 3.7 — Chạy app với PM2

**Lý do:** `dist/main.js` là file entry point sau khi build TypeScript → JavaScript. PM2 giữ process chạy liên tục.

```bash
pm2 start ~/backend/dist/main.js --name "backend"
pm2 startup   # tự khởi động khi reboot
pm2 save      # lưu danh sách process
```

## Bước 3.8 — Kiểm tra app chạy đúng

```bash
# Kiểm tra port đang lắng nghe
ss -tlnp | grep node

# Test API
curl http://localhost:8000
```