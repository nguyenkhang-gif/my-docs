# PART 7 — Common Commands & Important Notes

## Common Commands

### PM2

```bash
pm2 status                    # view status
pm2 logs backend              # view realtime logs
pm2 logs backend --lines 50   # view last 50 log lines
pm2 restart backend           # restart app
pm2 stop backend              # stop app
pm2 start ~/backend/dist/main.js --name "backend"  # start app
pm2 delete backend            # remove from PM2
pm2 save                      # save process list
pm2 startup                   # auto-start on reboot
```

### Nginx

```bash
sudo nginx -t                  # test config
sudo systemctl restart nginx   # restart nginx
sudo systemctl status nginx    # view status
```

### SSH

```bash
# Get current IP of your Mac
curl ifconfig.me

# SSH into server
ssh -i ~/.ssh/my-back-end-key-pair.pem ubuntu@YOUR_IP

# Fix REMOTE HOST IDENTIFICATION HAS CHANGED error
ssh-keygen -R YOUR_IP
```

### Server Monitoring

```bash
# CPU & RAM
nproc && free -h

# Disk
df -h

# View running ports
ss -tlnp | grep node

# Instance type
curl -s http://169.254.169.254/latest/meta-data/instance-type
```

### Manual Deploy (without GitHub Actions)

```bash
# On Mac — build and compress
npm run build
zip -r dist.zip dist/
scp -i ~/.ssh/my-back-end-key-pair.pem dist.zip ubuntu@YOUR_IP:~/backend/

# On server — extract and restart
cd ~/backend
rm -rf dist/
unzip dist.zip
rm dist.zip
pm2 restart backend
```

> 💡 Use `rsync` to only copy changed files (much faster from the second time onwards):

```bash
rsync -avz --progress -e "ssh -i ~/.ssh/my-back-end-key-pair.pem" ./dist ubuntu@YOUR_IP:~/backend/
```

---

## Important Notes

- **Elastic IP:** Always attach an Elastic IP — unattached ones are charged ~$3.6/month
- **`.pem` file:** Can only be downloaded once — if lost, you must create a new instance. Store at `~/.ssh/` with `chmod 400`
- **`.env` file:** Never commit to GitHub — create it manually on the server
- **RAM:** t2.micro only has 1GB — cannot build TypeScript directly on the server
- **Storage:** 8GB by default — node_modules takes up a lot, monitor usage
- **Free tier:** 750 hours/month or $100 credit — only run 1 instance at a time
- **Security Group:** Must open ports 22, 80, 443, 8000 — missing any port blocks external access
- **node_modules:** Never copy from Mac to Linux — always run `npm install` on the server
- **class-transformer / class-validator:** Must be in `dependencies`, not `devDependencies`
- **Supabase + Node.js 20:** Use `import * as ws from 'ws'` and pass `realtime: { transport: ws }` to `createClient`
- **Dynamic IP:** Your Mac's IP changes when you switch networks — update the SSH rule in Security Group accordingly
