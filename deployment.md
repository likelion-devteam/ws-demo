Hướng dẫn bạn các bước triển khai dự án NodeJS lên AWS EC2:

1. **Tạo EC2 Instance**

- Đăng nhập AWS Console
- Truy cập EC2 Dashboard
- Click "Launch Instance"
- Chọn Amazon Linux 2023 AMI
- Chọn t2.micro (free tier)
- Tạo hoặc chọn key pair để SSH
- Mở port 22 (SSH) và 433 (HTTPs) 80 (HTTP) trong Security Group
- Launch instance

2. **Kết nối SSH vào EC2**

```bash
# Sử dụng file .pem đã tải về
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

3. **Cài đặt môi trường trên EC2**

```bash
# Cập nhật hệ thống
sudo yum update -y

# Cài đặt Git
sudo yum install git -y

# Cài đặt NodeJS
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 16  # hoặc version NodeJS bạn cần

# Kiểm tra cài đặt
git --version
node --version
npm --version
```

4. **Clone và cài đặt project**

```bash
# Clone project từ repository (nếu có)
git clone your-repository-url
# Hoặc tạo thư mục và upload code
mkdir app
cd app
# (Upload code từ máy local lên)

# Cài đặt dependencies
npm install

# Kiểm tra ứng dụng chạy được không
node index.js
```

5. **Cài đặt PM2 để quản lý process**

```bash
# Cài đặt PM2
npm install -g pm2

# Chạy ứng dụng với PM2
pm2 start index.js --name "node-app"

# Một số lệnh PM2 hữu ích
pm2 status          # Xem trạng thái
pm2 logs           # Xem logs
pm2 restart all    # Khởi động lại
pm2 startup        # Tự động chạy khi server reboot
```

6. **Cấu hình Nginx (Optional - nếu muốn dùng reverse proxy)**

```bash
# Cài đặt Nginx
sudo yum install nginx -y

# Tạo file cấu hình
sudo nano /etc/nginx/conf.d/node-app.conf

# Thêm cấu hình sau
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# Khởi động Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

7. **Các lưu ý quan trọng**:

- Cấu hình Security Group cho phép traffic vào port cần thiết

8. **Một số lệnh hữu ích để quản lý**:

```bash
# Xem logs của ứng dụng
pm2 logs

# Kiểm tra memory usage
free -m

# Kiểm tra disk usage
df -h

# Kiểm tra process đang chạy
ps aux | grep node

# Restart ứng dụng
pm2 restart all
```

Để kiểm tra ứng dụng đã hoạt động, truy cập:

- http://your-ec2-public-ip:3000 (nếu dùng port mặc định của Node)
