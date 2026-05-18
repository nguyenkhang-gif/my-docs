# PHẦN 4 — Cấu hình Nginx

**Lý do:** Nginx nhận request từ port 80 và chuyển vào NestJS đang chạy ở port 8000. Đây gọi là reverse proxy.

```bash
sudo nano /etc/nginx/sites-available/my-backend
```

```nginx
server {
    listen 80;
    server_name YOUR_ELASTIC_IP;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/my-backend /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t           # kiểm tra config
sudo systemctl restart nginx
```
