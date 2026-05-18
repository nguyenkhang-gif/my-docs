# PART 1 — Setting Up AWS EC2

## Step 1.1 — Launch EC2 Instance

**Why:** EC2 is a virtual server on AWS where your backend runs 24/7.

Go to AWS Console → EC2 → Launch Instance:

- **AMI:** Ubuntu Server 22.04 LTS
- **Instance type:** t2.micro (free tier, 1 vCPU, 1GB RAM)
- **Key pair:** Create new, download the `.pem` file (can only be downloaded once!)
- **Security Group:** Open port 22 (SSH) for now, more ports added in Step 1.3

> ⚠️ The `.pem` file usually downloads to `~/Downloads`, move it to `~/.ssh/`:

```bash
mv ~/Downloads/my-back-end-key-pair.pem ~/.ssh/
chmod 400 ~/.ssh/my-back-end-key-pair.pem
```

## Step 1.2 — Attach Elastic IP

**Why:** By default, EC2's IP changes every restart. Elastic IP gives you a fixed IP that never changes no matter how many times you restart.

EC2 Console → Elastic IPs → Allocate → Associate with the instance you just created.

> ⚠️ If you have an Elastic IP but **don't attach** it to an instance, you'll be charged ~$3.6/month.

## Step 1.3 — Configure Security Group

**Why:** Security Group is AWS's firewall, controlling which ports are accessible from outside.

EC2 → Security Groups → launch-wizard-1 → Inbound rules → Edit inbound rules → Add rule:

| Port | Protocol | Source | Used for |
|------|----------|--------|----------|
| 22 | TCP | Your IP/32 | SSH into server |
| 80 | TCP | 0.0.0.0/0 | HTTP (Nginx) |
| 443 | TCP | 0.0.0.0/0 | HTTPS |
| 8000 | TCP | 0.0.0.0/0 | NestJS API |

> 💡 To restrict SSH to only your IP (more secure), get your current IP:

```bash
curl ifconfig.me
# Example: 1.55.201.80 → enter 1.55.201.80/32 as Source for port 22
```

> ⚠️ Dynamic IPs can change when you switch networks. If you can't SSH in, re-run the command above and update the rule.

## Step 1.4 — Create Directories on Server

**Why:** Separate FE and BE directories for easier management and to avoid conflicts.

```bash
mkdir -p ~/frontend ~/backend
```
