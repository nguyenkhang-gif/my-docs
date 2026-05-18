# PART 4 — Nginx Configuration

**Why:** Nginx receives requests on port 80 and forwards them to NestJS running on port 8000. This is called a reverse proxy.

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
sudo nginx -t           # test config
sudo systemctl restart nginx
```
