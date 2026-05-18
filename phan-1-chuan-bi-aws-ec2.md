# PHẦN 1 — Chuẩn bị AWS EC2

## Bước 1.1 — Tạo EC2 Instance

**Lý do:** EC2 là máy chủ ảo trên AWS, nơi chạy backend của bạn 24/7.

Vào AWS Console → EC2 → Launch Instance:

- **AMI:** Ubuntu Server 22.04 LTS
- **Instance type:** t2.micro (free tier, 1 vCPU, 1GB RAM)
- **Key pair:** Tạo mới, download file `.pem` về máy (chỉ download được 1 lần!)
- **Security Group:** Tạm thời chỉ mở port 22 (SSH), sẽ mở thêm ở Bước 1.3

> ⚠️ File `.pem` thường download vào thư mục `~/Downloads`, cần chuyển vào `~/.ssh/`:

```bash
mv ~/Downloads/my-back-end-key-pair.pem ~/.ssh/
chmod 400 ~/.ssh/my-back-end-key-pair.pem
```

## Bước 1.2 — Gắn Elastic IP

**Lý do:** Mặc định IP của EC2 thay đổi mỗi lần restart. Elastic IP giúp IP cố định, không đổi dù restart bao nhiêu lần.

EC2 Console → Elastic IPs → Allocate → Associate với instance vừa tạo.

> ⚠️ Nếu có Elastic IP mà **không gắn** vào instance sẽ bị tính phí ~$3.6/tháng.

## Bước 1.3 — Mở Security Group

**Lý do:** Security Group là firewall của AWS, kiểm soát port nào được phép truy cập từ bên ngoài.

EC2 → Security Groups → launch-wizard-1 → Inbound rules → Edit inbound rules → Add rule:

|Port|Protocol|Source|Dùng cho|
|---|---|---|---|
|22|TCP|IP của bạn/32|SSH vào server|
|80|TCP|0.0.0.0/0|HTTP (Nginx)|
|443|TCP|0.0.0.0/0|HTTPS|
|8000|TCP|0.0.0.0/0|NestJS API|

> 💡 Để giới hạn SSH chỉ từ IP của bạn (bảo mật hơn), lấy IP hiện tại:

```bash
curl ifconfig.me
# Ví dụ: 1.55.201.80 → nhập 1.55.201.80/32 vào Source của port 22
```

> ⚠️ IP động có thể thay đổi khi đổi mạng. Khi không SSH được, chạy lại lệnh trên và update rule mới.

## Bước 1.4 — Tạo thư mục trên server

**Lý do:** Tách riêng FE và BE để dễ quản lý, tránh conflict.

```bash
mkdir -p ~/frontend ~/backend
```