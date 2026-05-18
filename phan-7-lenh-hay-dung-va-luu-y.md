# PHẦN 7 — Các lệnh hay dùng & Lưu ý quan trọng

## Các lệnh hay dùng

### PM2

```bash
pm2 status                    # xem trạng thái
pm2 logs backend              # xem logs realtime
pm2 logs backend --lines 50   # xem 50 dòng log gần nhất
pm2 restart backend           # restart app
pm2 stop backend              # dừng app
pm2 start ~/backend/dist/main.js --name "backend"  # start app
pm2 delete backend            # xóa khỏi PM2
pm2 save                      # lưu danh sách process
pm2 startup                   # tự khởi động khi reboot
```

### Nginx

```bash
sudo nginx -t                  # kiểm tra config
sudo systemctl restart nginx   # restart nginx
sudo systemctl status nginx    # xem trạng thái
```

### SSH

```bash
# Lấy IP hiện tại của máy Mac
curl ifconfig.me

# SSH vào server
ssh -i ~/.ssh/my-back-end-key-pair.pem ubuntu@YOUR_IP

# Xử lý lỗi REMOTE HOST IDENTIFICATION HAS CHANGED
ssh-keygen -R YOUR_IP
```

### Kiểm tra server

```bash
# CPU & RAM
nproc && free -h

# Disk
df -h

# Xem port đang chạy
ss -tlnp | grep node

# Instance type
curl -s http://169.254.169.254/latest/meta-data/instance-type
```

### Deploy thủ công (không dùng GitHub Actions)

```bash
# Trên Mac — build và nén
npm run build
zip -r dist.zip dist/
scp -i ~/.ssh/my-back-end-key-pair.pem dist.zip ubuntu@YOUR_IP:~/backend/

# Trên server — giải nén và restart
cd ~/backend
rm -rf dist/
unzip dist.zip
rm dist.zip
pm2 restart backend
```

> 💡 Dùng `rsync` để chỉ copy file thay đổi (nhanh hơn lần 2 trở đi):

```bash
rsync -avz --progress -e "ssh -i ~/.ssh/my-back-end-key-pair.pem" ./dist ubuntu@YOUR_IP:~/backend/
```

---

## Lưu ý quan trọng

- **Elastic IP:** Luôn gắn Elastic IP, không gắn sẽ bị tính phí ~$3.6/tháng
- **File .pem:** Chỉ download được 1 lần, mất là phải tạo instance mới. Lưu ở `~/.ssh/` và `chmod 400`
- **File .env:** Không commit lên GitHub, tạo thủ công trên server
- **RAM:** t2.micro chỉ 1GB, không build TypeScript trực tiếp trên server được
- **Storage:** 8GB mặc định, node_modules chiếm khá nhiều, cần theo dõi
- **Free tier:** 750 giờ/tháng hoặc $100 credit, chỉ nên chạy 1 instance
- **Security Group:** Phải mở đủ port 22, 80, 443, 8000 — thiếu port nào là không truy cập được từ ngoài
- **node_modules:** Không copy từ Mac lên Linux — phải `npm install` lại trên server
- **class-transformer/class-validator:** Để trong `dependencies` không phải `devDependencies`
- **Supabase + Node.js 20:** Dùng `import * as ws from 'ws'` và truyền `realtime: { transport: ws }` vào `createClient`
- **IP động:** IP máy Mac thay đổi khi đổi mạng, cần update lại SSH rule trong Security Group