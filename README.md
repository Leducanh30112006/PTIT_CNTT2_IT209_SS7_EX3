# Báo Cáo Bài 3: Thiết lập Cơ sở dữ liệu và Tự cấu hình dịch vụ Systemd cho Spring Boot

## 1. Thiết lập Cơ sở dữ liệu MySQL và Người dùng

### 1.1. Khởi tạo Database và User trong MySQL
```sql
CREATE DATABASE springboot_db;
CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
FLUSH PRIVILEGES;
```

### 1.2. Tạo người dùng hệ thống Linux `spring-runner`
```bash
sudo useradd -r -s /usr/sbin/nologin spring-runner
```

---

## 2. Cấu hình dịch vụ Systemd (`/etc/systemd/system/spring-app.service`)

Nội dung tệp cấu hình:
```ini
[Unit]
Description=Spring Boot Application Service
After=network.target

[Service]
Type=simple
User=spring-runner
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## 3. Khởi chạy và Kiểm tra dịch vụ

### 3.1. Nạp cấu hình và khởi động dịch vụ
```bash
sudo systemctl daemon-reload
sudo systemctl start spring-app.service
```

### 3.2. Kiểm tra trạng thái dịch vụ (`systemctl status spring-app.service`)
```bash
$ sudo systemctl status spring-app.service
● spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-10-08 14:30:15 UTC; 12s ago
   Main PID: 18452 (java)
      Tasks: 28 (limit: 4613)
     Memory: 245.8M
     CGroup: /system.slice/spring-app.service
             └─18452 /usr/bin/java -jar /opt/spring-app/app.jar

Oct 08 14:30:15 ubuntu-server systemd[1]: Started Spring Boot Application Service.
Oct 08 14:30:22 ubuntu-server java[18452]: Tomcat started on port(s): 8082 (http) with context path ''
Oct 08 14:30:22 ubuntu-server java[18452]: Started Application in 6.42 seconds (JVM running for 7.15)
```

### 3.3. Kiểm tra cổng lắng nghe (`ss -tlnp | grep 8082`)
```bash
$ ss -tlnp | grep 8082
LISTEN 0      100          *:8082            *:*    users:(("java",pid=18452,fd=42))
```

**Kết luận:** Dịch vụ `spring-app.service` chạy thành công dưới user `spring-runner`, kết nối CSDL MySQL và đang lắng nghe ổn định tại cổng `8082`.
