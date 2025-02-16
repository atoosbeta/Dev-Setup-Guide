# Document Dev

> Hướng Dẫn Thiết Lập Môi Trường Phát Triển (Dev Setup Guide)

## Guide NestJs, Redis, Docker 
1. NestJs
-  Chạy một resource (router) trong NestJS
```bash
nest g resource <tên router> --no-spec
```
2. Redis
-  Kết nối Redis
```bash
redis-cli -h <hostname_or_IP> -p 6379 -a <password>
```
3. Docker
-  Xóa tất cả Docker images
```bash
docker rmi -f $(docker images -q)
```
-  Xóa tất cả các container, bao gồm cả các container đang chạy
```bash
docker rm -f $(docker ps -aq)
```
-  Tạo một mạng Docker mới
```bash
docker network create <tên mạng>
```
-  Kết nối container vào mạng Docker
```bash
docker network connect <tên mạng> <tên container>
```
-  Kiểm tra các kết nối mạng Docker
```bash
docker network inspect <tên mạng>
```

## Setup NVM
1. Cài đặt NVM trên Linux/MacOS
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```
2. Tải lại terminal
```bash
source ~/.bashrc
```
3. Kiểm tra cài đặt
```bash
nvm --version
```

## Setup SSH key
1. Điều Hướng Đến Thư Mục SSH
```bash
cd ~/.ssh
```
2. Cấu Hình Tài Khoản Git
```bash
git config --global user.name "Họ Tên"
git config --global user.email "example@gmail.com"
```
3. Kiểm Tra Thông Tin Tài Khoản Đã Cấu Hình
```bash
git config --global --list
```
4. Tạo SSH Key
```bash
ssh-keygen -t rsa -b 4096 -C "example@gmail.com"
```
5. Copy SSH Key Để Thêm Vào GitHub/GitLab
```bash
cat ~/.ssh/id_rsa.pub
```

## Setup Nginx trên Linux
-  Điều Hướng Đến Thư Mục sites-available
```bash
cd /etc/nginx/sites-available
```
-  Điều Hướng Đến Thư Mục www
```bash
cd /var/www/
```

1. Cài đặt Nginx
```bash
sudo apt install nginx
```
2. Kiểm tra Trạng thái của Nginx
```bash
sudo apt install nginx
```
3. Tạo một thư mục mới cho ứng dụng của bạn (ví dụ: /var/www/myapp)
```bash
sudo mkdir -p /var/www/myapp/html
sudo chown -R $USER:$USER /var/www/myapp/html
```
4. Tạo một file cấu hình cho server block trong thư mục
```bash
sudo nano /etc/nginx/sites-available/myapp
```
5. Nội dung file cấu hình FrontEnd
```yaml
server {
    listen 80;
    server_name <domain>;

    root /var/www/myapp/html; # Đường dẫn đúng tới thư mục chứa tệp tĩnh (HTML, CSS, JS)
    index index.html;

    # Cấu hình cho việc phục vụ ứng dụng ReactJS (SPA)
    location / {
        try_files $uri /index.html;  # chuyển hướng các yêu cầu không tìm thấy tệp vào index.html
    }

    # Cấu hình cache cho các tệp tĩnh (JS, CSS, hình ảnh, v.v.) để tối ưu hiệu suất
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        expires 30d;  # Các tệp tĩnh sẽ được cache trong 30 ngày
        add_header Cache-Control "public, no-transform";  # Thêm header cache cho tệp tĩnh
    }

    # Cấu hình bảo mật thêm cho Nginx (bảo vệ khỏi các tấn công cơ bản)
    location / {
        try_files $uri $uri/ =404;  # Nếu không tìm thấy tệp, trả về lỗi 404
    }

    # Nếu bạn sử dụng HTTPS, bạn có thể cấu hình SSL ở đây.
    # Ví dụ:
    # listen 443 ssl;
    # ssl_certificate /etc/nginx/ssl/myapp.crt;
    # ssl_certificate_key /etc/nginx/ssl/myapp.key;
}
```
6. Nội dung file cấu hình BackEnd
```yaml
server {
    listen 80;
    server_name <domain>;

    location / {
        proxy_pass http://localhost:<port>; # Địa chỉ của ứng dụng của bạn
        proxy_http_version 1.1;

        # Các header cần thiết khi proxy HTTP
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;

        # Đảm bảo không cache các yêu cầu nâng cấp (websocket)
        proxy_cache_bypass $http_upgrade;

        # Các header để duy trì thông tin IP gốc và các thông tin khác
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

    }
}
```
7. Kích hoạt server block bằng cách tạo một symbolic link đến thư mục sites-enabled
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled
```
8. Kiểm tra cấu hình Nginx để đảm bảo không có lỗi
```bash
sudo nginx -t
```
9. Tải lại Nginx để áp dụng cấu hình mới
```bash
sudo systemctl reload nginx
```

## Setup PM2
1. Cài đặt PM2
```bash
npm install -g pm2
```
2. Kiểm tra lại PM2 
```bash
pm2 -v
```
3. Để chạy một script dev bằng npm với PM2
```bash
pm2 start npm --name=website -- run dev
```
4. Để chạy một script bằng yarn với PM2
```bash
pm2 start --name=website yarn -- start
```

-   Dừng ứng dụng
```bash
pm2 stop <id>
```
-   Khởi động lại ứng dụng
```bash
pm2 restart <id>
```
-   Xóa ứng dụng khỏi danh sách quản lý
```bash
pm2 delete <id>
```
-   Xem log của ứng dụng
```bash
pm2 logs <id>
```
-   Xem thông tin chi tiết của ứng dụng
```bash
pm2 describe <id>
```
-   Chạy lệnh sau để thiết lập autostart
```bash
pm2 startup
```
-   Lệnh này sẽ tạo một lệnh đầu ra cho bạn, giống như
```bash
sudo env PATH=$PATH:/usr/bin pm2 startup ubuntu -u <username> --hp /home/<username>
```
-   Chạy lệnh này để hoàn tất việc thiết lập
```bash
sudo env PATH=$PATH:/usr/bin pm2 startup ubuntu -u <username> --hp /home/<username>
```
-   Lưu lại các tiến trình của PM2, sau khi đã thiết lập autostart
```bash
pm2 save
```

## Setup sequelize
- Cài đặt sequelize
```bash
npm install sequelize sequelize-cli
```
### Create, Run, Undo Migrations in Sequelize
1. Tạo một migration (bảng) mới
```bash
npx sequelize-cli migration:generate --name <tên file>
```
2. Chạy migration cho một bảng cụ thể
```bash
npx sequelize-cli db:migrate --name <tên file>
```
3. Chạy tất cả migration để tạo bảng vào database
```bash
npx sequelize-cli db:migrate
```
4. Undo migration cho một bảng cụ thể (hoàn tác tạo bảng)
```bash
npx sequelize-cli db:migrate:undo --name <tên file>
```
5. Undo tất cả các migration (hoàn tác tất cả migration đã chạy)
```bash
npx sequelize-cli db:migrate:undo
```

### Create, Run, Undo Seeder in Sequelize
1. Tạo một file seeder
```bash
npx sequelize-cli seed:generate --name <tên file>
```
2. Chạy một file seeder
```bash
npx sequelize-cli db:seed --seed <tên file>
```
3. Undo một file seeder
```bash
npx sequelize-cli db:seed:undo --seed <tên file>
```
4. Chạy tất cả file seeder
```bash
npx sequelize-cli db:seed:all
```
5. Undo tất cả file seeder (hoàn tác tất cả seeder đã chạy)
```bash
npx sequelize-cli db:seed:undo:all
```
