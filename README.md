# Bài 5: Tự động hóa điều phối đa dịch vụ với Docker Compose & File .env

## 1. Mục tiêu
- Đóng gói cụm 2 dịch vụ (Backend + Database PostgreSQL) thành tệp khai báo `docker-compose.yml` theo chuẩn Infrastructure as Code (IaC).
- Tách và bảo mật các biến môi trường nhạy cảm bằng file `.env` và cung cấp tệp mẫu `.env.example`.
- Khai báo đầy đủ các chỉ thị `depends_on`, `networks`, `volumes` ở cấp độ root.
- Quản trị toàn bộ vòng đời ứng dụng bằng Docker Compose CLI v2 (`docker compose`).

---

## 2. Cấu trúc thư mục
```text
Bai5/
├── .env.example        # File mẫu cấu hình biến môi trường
├── .env                # File biến môi trường thực tế (được ignore qua .gitignore)
├── .gitignore          # Chặn commit file .env chứa thông tin nhạy cảm
├── docker-compose.yml  # File điều phối đa container (IaC)
└── README.md           # Hướng dẫn và báo cáo kết quả thực hiện
```

---

## 3. Cấu hình biến môi trường

### File `.env.example`:
```env
DB_NAME=your_db_name
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_PORT=5432
PORT=8080
```

### File `.env` (sử dụng khi chạy):
```env
DB_NAME=testdb
DB_USER=myuser
DB_PASSWORD=mypassword
DB_PORT=5432
PORT=8080
```

---

## 4. Các bước thực hiện & Kiểm tra với Docker Compose v2

### Bước 1: Khởi chạy cụm dịch vụ ở chế độ chạy ngầm
```bash
docker compose up -d
```
**Kết quả:**
```text
[+] up 4/4
 ✔ Network root_backend-net Created     0.1s
 ✔ Volume root_pgdata       Created     0.0s
 ✔ Container db-server      Started     0.7s
 ✔ Container app-backend    Started     0.7s
```

### Bước 2: Kiểm tra trạng thái các dịch vụ
```bash
docker compose ps
```
**Kết quả:**
```text
NAME          IMAGE                COMMAND                  SERVICE    CREATED          STATUS          PORTS
app-backend   alpine               "sh -c 'apk add --no…"   backend    18 seconds ago   Up 18 seconds
db-server     postgres:15-alpine   "docker-entrypoint.s…"   database   18 seconds ago   Up 18 seconds   5432/tcp
```
*(Cả 2 dịch vụ `app-backend` và `db-server` đều ở trạng thái `Up`).*

### Bước 3: Theo dõi logs của các dịch vụ
```bash
docker compose logs -f
```
*(Hiển thị logs khởi tạo của `db-server` sẵn sàng nhận kết nối tại cổng 5432 và `app-backend` cài đặt các gói công cụ mạng).*

### Bước 4: Kiểm tra kết nối nội bộ (DNS Resolution) giữa Backend và Database
```bash
docker compose exec backend ping -c 2 database
```
**Kết quả:**
```text
PING database (172.19.0.2) 56(84) bytes of data.
64 bytes from db-server.root_backend-net (172.19.0.2): icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from db-server.root_backend-net (172.19.0.2): icmp_seq=2 ttl=64 time=0.105 ms

--- database ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.075/0.090/0.105/0.015 ms
```
*(Container `backend` phân giải và giao tiếp thành công với `database` qua hostname).*

### Bước 5: Dừng và dọn dẹp cụm dịch vụ
```bash
docker compose down
```
**Kết quả:**
```text
[+] down 3/3
 ✔ Container app-backend    Removed     10.3s
 ✔ Container db-server      Removed      0.4s
 ✔ Network root_backend-net Removed      0.2s
```
