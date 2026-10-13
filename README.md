# Campus Hub

**Student name:** Nguyễn Quang Vinh  
**Student ID:** B24DTCN452

---

## Giới thiệu

Campus Hub là ứng dụng Spring Boot + MySQL cho phép sinh viên đăng bài, tìm đồ thất lạc, hỏi đáp, trao đổi đồ dùng và tìm nhóm học tập.

---

## Yêu cầu

| Công cụ | Phiên bản tối thiểu |
|---|---|
| Docker | 24.x trở lên |
| Docker Compose (plugin) | v2.x (`docker compose`) |
| Java 21 JDK | Chỉ cần khi chạy test hoặc build ngoài Docker |

---

## Phần 1 — Build Docker image

### Bước 1: Build JAR trên máy host

```bash
cd backend
chmod +x gradlew
./gradlew bootJar -x test
```

Kết quả: `backend/build/libs/campus-hub-1.0.0.jar`

### Bước 2: Build Docker image

Chạy từ **thư mục gốc** của project (nơi chứa `compose.yml`):

```bash
docker build -t campus-hub-backend:latest ./backend
```

### Bước 3: Kiểm tra image đã tồn tại

```bash
docker images campus-hub-backend
```

Kết quả mong đợi:

```
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
campus-hub-backend     latest    <id>           <time>          <size>
```

### Screenshot 1 — Docker build thành công và `docker images`

> **[CHỤP MÀN HÌNH]** Sau khi chạy `docker build` và `docker images`, chụp terminal
> hiển thị output build kết thúc bằng `FINISHED` và bảng `docker images` có dòng
> `campus-hub-backend   latest`.
>
> Lưu ảnh vào `screenshots/01_docker_build.png` và thay thế khối này bằng:
> `![Docker build và docker images](screenshots/01_docker_build.png)`

---

## Phần 2 — Chạy backend và MySQL bằng Docker Compose

### Khởi động services

```bash
docker compose up -d --build
```

Lệnh này sẽ:
1. Pull image `mysql:8.4` (lần đầu).
2. Build image `campus-hub-backend:latest` từ `./backend/Dockerfile`.
3. Khởi động MySQL, chờ đến khi healthy, sau đó mới khởi động backend.

### Screenshot 2 — Các services đang chạy

```bash
docker compose ps
```

> **[CHỤP MÀN HÌNH]** Chụp output của `docker compose ps` hiển thị cả hai service
> `mysql` và `backend` với trạng thái `running`.
>
> Lưu ảnh vào `screenshots/02_compose_ps.png` và thay thế khối này bằng:
> `![Compose services đang chạy](screenshots/02_compose_ps.png)`

---

## Kiểm tra trạng thái và logs

```bash
# Xem trạng thái tất cả service
docker compose ps

# Xem log backend (100 dòng cuối)
docker compose logs --tail=100 backend

# Xem log MySQL
docker compose logs --tail=50 mysql
```

---

## Health endpoint

Sau khi backend khởi động, kiểm tra endpoint health:

```bash
curl -i http://localhost:8080/actuator/health
```

Kết quả mong đợi:

```
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json

{"status":"UP","components":{"db":{"status":"UP"},...}}
```

### Screenshot 3 — HTTP 200 và `{"status":"UP"}`

> **[CHỤP MÀN HÌNH]** Chụp output của lệnh `curl` hiển thị `HTTP/1.1 200 OK` và
> JSON body chứa `"status":"UP"`.
>
> Lưu ảnh vào `screenshots/03_health.png` và thay thế khối này bằng:
> `![Health endpoint trả về 200 UP](screenshots/03_health.png)`

---

## Dừng services

```bash
# Dừng và xóa containers (giữ nguyên volume database)
docker compose down

# Dừng, xóa containers VÀ xóa volume database (reset hoàn toàn)
docker compose down -v
```

---

## Biến môi trường

Ứng dụng đọc các biến sau (giá trị mặc định trong ngoặc):

| Biến | Mặc định | Mô tả |
|---|---|---|
| `DB_URL` | `jdbc:mysql://localhost:3306/campus_hub?createDatabaseIfNotExist=true` | JDBC URL kết nối DB |
| `DB_USERNAME` | `root` | Tên đăng nhập database |
| `DB_PASSWORD` | `123456` | Mật khẩu database |
| `SERVER_PORT` | `8080` | Cổng HTTP |
| `CORS_ALLOWED_ORIGINS` | `http://localhost:5173` | CORS origin cho frontend |

Trong `compose.yml`, các biến này được ghi đè để backend kết nối đến service `mysql` trong Compose network.

---

## Tài khoản demo (được tạo tự động khi khởi động lần đầu)

| Vai trò | Tên đăng nhập | Mật khẩu |
|---|---|---|
| Sinh viên | `student` | `student123` |
| Quản trị viên | `admin` | `admin123` |

> **Lưu ý:** Các tài khoản này chỉ dùng cho môi trường phát triển local.

---

## Cấu trúc files Docker đã tạo

```
campus-hub-base-main/
├── compose.yml                  ← Docker Compose (backend + mysql)
├── README.md                    ← File này
└── backend/
    ├── Dockerfile               ← Multi-stage build (JDK builder → JRE runtime)
    └── .dockerignore            ← Loại trừ files không cần thiết
```

---

## API chính

| Method | Path | Xác thực |
|---|---|---|
| `POST` | `/api/auth/register` | Không |
| `GET` | `/api/auth/me` | HTTP Basic |
| `GET` | `/api/posts` | Không |
| `POST` | `/api/posts` | HTTP Basic |
| `GET` | `/actuator/health` | Không |

---

## Chạy test

```bash
cd backend
chmod +x gradlew
./gradlew test
```

Kết quả mong đợi: `BUILD SUCCESSFUL` với 4 tests passed.

---

## Git commands để review và commit

```bash
# Xem các file đã thay đổi
git status
git diff

# Stage và commit
git add backend/Dockerfile backend/.dockerignore compose.yml README.md
git commit -m "feat: add Docker and Docker Compose configuration"

# Push (khi cần)
git push origin main
```
