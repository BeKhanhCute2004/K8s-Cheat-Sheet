# 🐳 Docker Ultimate Cheat Sheet

## 🏗️ 0. Container Runtime (Kiến thức nền)
Container Runtime là thành phần chịu trách nhiệm chạy container.
* **High-level (containerd, CRI-O):** Quản lý vòng đời Image, truyền tải dữ liệu, quản lý Network và Storage.
* **Low-level (runc):** Làm việc trực tiếp với Kernel (Namespaces, Cgroups) để tạo và chạy container.

---

## 📦 1. Thao tác với Container
### Khởi tạo và Vận hành
* `docker container create/start/run [tên image]`
* **Flags phổ biến:**
    * `-it`: Chế độ tương tác (Interactive) và Terminal (TTY).
    * `-d`: Chạy ngầm (Detached).
    * `-a`: Attach vào container đang chạy.
    * `--name=[tên]`: Đặt tên định danh cho container.
    * `--rm`: Tự động xóa container khi dừng.
    * `--restart`: Chính sách khởi động lại:
        * `no`: (Mặc định) Không tự động restart.
        * `on-failure[:max-retries]`: Chỉ restart khi lỗi (exit code != 0).
        * `always`: Luôn restart (kể cả khi reboot máy host).
        * `unless-stopped`: Luôn restart trừ khi bị người dùng `stop` thủ công.
    * `-p [IP_mặt_ngoài]:[port_ngoài]:[port_container]`: Ánh xạ cổng.
    * `--expose=[port]`: Khai báo thêm cổng nội bộ.
    * `--network=[bridge|none|host|name]`: Thiết lập mạng.

### Quản lý Dữ liệu (Mounts)
* **Volume:** `--mount type=volume,source=[tên_volume],target=/data` (Thêm `,readonly` nếu cần).
* **Bind Mount:** `--mount type=bind,source=/opt/data,target=/app/data` (Liên kết trực tiếp thư mục host).
* **Tmpfs (Lưu trên RAM):** * `--mount type=tmpfs,target=/app/tmp,tmpfs-size=64m` (Giới hạn dung lượng).
    * `--mount type=tmpfs,target=/app/tmp,tmpfs-mode=1777` (Quyền 777 kèm **Sticky Bit** - chỉ chủ sở hữu mới xóa được file của mình).

### Kiểm tra và Giám sát
* `docker container ls -lqa`: Liệt kê (l: mới nhất, q: chỉ ID, a: tất cả).
* `docker container exec -it [name] [COMMAND]`: Thực thi lệnh (thường dùng `/bin/bash`).
* `docker container inspect [ID]`: Xem chi tiết cấu hình JSON.
* `docker container stats`: Theo dõi tài nguyên (CPU, RAM, Disk I/O).
* `docker container top [ID]`: Xem các tiến trình bên trong container.
* `docker container logs [ID]`: Xem nhật ký hoạt động.
* `docker system events`: Xem các sự kiện thời gian thực từ server.

### Quản trị
* `docker container kill [ID] --signal=9`: Dừng khẩn cấp (9: SIGKILL, 15: SIGTERM).
* `docker container rm [ID]`: Xóa container.
* `docker container prune`: Xóa sạch các container đã dừng.
* `docker container cp [SRC] [DEST]`: Copy file (Lưu ý format `container_id:path`).
* `docker container ps / port <tên>`: Kiểm tra trạng thái và cổng.
* `docker container rename [old] [new]`: Đổi tên container.
* `docker container commit -m "msg" -a "author" [image:tag]`: Đóng gói container thành image.

---

## 🖼️ 2. Thao tác với Image
* `docker image ls`: Liệt kê các image hiện có.
* `docker search [image] --limit=5 --filter stars=10`: Tìm kiếm image trên Hub.
* `docker image pull/push [registry]/[acc]/[repo]:[tag]`: Tải/Đẩy image.
* `docker image tag [old_tag] [new_tag]`: Gắn thẻ mới cho image.
* `docker image rm [image:tag]`: Xóa image.
* `docker image prune -a`: Xóa toàn bộ image không sử dụng.
* `docker image history [image]`: Xem các lớp (layers) tạo nên image.
* `docker image inspect -f '{{ .Config.Env }}' [image]`: Trích xuất thông tin cụ thể (dấu `.` là gốc JSON).
* `docker build -t [name:tag] -f [Dockerfile_path] [context]`: Build image từ Dockerfile.

### Bảng so sánh xuất/nhập:
| Lệnh | Đối tượng | Kết quả | Giữ Layers/History |
| :--- | :--- | :--- | :--- |
| **commit** | Container → Image | Image mới | ✓ Có |
| **save** | Image → File.tar | File tar | ✓ Có |
| **export** | Container → File.tar | File tar | ✗ Không (Flatten) |

---

## 🌐 3. Docker Network
* `docker network ls`: Xem danh sách network.
* `docker network create --driver=[bridge|none|host] --subnet=[CIDR] [name]`: Tạo mạng.
* `docker network inspect [name]`: Xem chi tiết các container trong mạng.
* `docker network connect/disconnect [net] [container]`: Gắn/Tháo mạng khỏi container.
* `docker network rm/prune`: Xóa mạng.

---

## 💾 4. Docker Volume
* `docker volume create --driver=local [name]`: Tạo vùng lưu trữ.
* `docker volume ls / inspect / rm / prune`: Quản lý volume.

---

## 🐙 5. Docker Compose
* `docker-compose up -d`: Khởi chạy dịch vụ.
* `docker-compose down`: Dừng và xóa tài nguyên.
* `docker-compose ps / logs -f / restart`: Quản lý trạng thái.

### Template `docker-compose.yml`:
```yaml
version: "3.9"
services:
  web_app:
    build:
      context: ./app
      dockerfile: Dockerfile.dev
    container_name: my_web
    ports:
      - "80:8000"
    volumes:
      - ./src:/app/src:ro            # Bind mount
      - web_logs:/var/log/app        # Named volume
    environment:
      - DEBUG=true
    env_file: .env.production
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend_net
      - backend_net
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend_net
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
  web_logs:

networks:
  frontend_net:
  backend_net:
    internal: true
```

## ⚙️ 6. Quản lý Tài nguyên (Resource Constraints - Cgroups)

Về bản chất, Docker không tự giới hạn tài nguyên mà nó yêu cầu Linux Kernel sử dụng **Cgroups** để kiểm soát các tiến trình. Dưới đây là các cờ quan trọng để cấu hình hạn ngạch:

### 🧠 Bộ nhớ (Memory)
| Flag | Giải thích |
| :--- | :--- |
| `--memory` (hoặc `-m`) | **Hard limit:** Giới hạn RAM tối đa. Nếu container vượt quá, nó sẽ bị Kernel tiêu diệt (OOM Kill). |
| `--memory-reservation` | **Soft limit:** Giới hạn đảm bảo. Container có thể dùng quá mức này khi máy rảnh, nhưng khi hệ thống thiếu RAM, Kernel sẽ ép container lùi về mức này. |
| `--memory-swap` | Tổng lượng **RAM + Swap** khả dụng. Ví dụ: `--memory=512m --memory-swap=1g` nghĩa là container có 512MB RAM và 512MB Swap. |

### ⚡ Vi xử lý (CPU)
| Flag | Giải thích |
| :--- | :--- |
| `--cpus` | Giới hạn số lượng nhân CPU. Ví dụ: `0.5` (nửa nhân), `2.0` (2 nhân). |
| `--cpu-shares` | **Trọng số ưu tiên (Default: 1024):** Khi các container tranh chấp CPU, hệ thống chia tài nguyên theo tỉ lệ của con số này. |
| `--cpuset-cpus` | **Cố định nhân:** Chỉ cho phép container chạy trên các nhân cụ thể. Ví dụ: `0,1` (nhân 0 và 1), `0-3` (nhân 0 đến 3). |



### 💾 Ổ đĩa (Block I/O)
Giới hạn tốc độ đọc/ghi (Thường áp dụng cho thiết bị cụ thể như `/dev/sda`).
| Flag | Giải thích |
| :--- | :--- |
| `--device-read-bps` | Giới hạn băng thông đọc (Bytes/giây). VD: `--device-read-bps /dev/sda:10mb`. |
| `--device-write-bps` | Giới hạn băng thông ghi (Bytes/giây). VD: `--device-write-bps /dev/sda:10mb`. |
| `--device-read-iops` | Giới hạn số lượng tác vụ đọc/giây (IOPS). |
| `--device-write-iops` | Giới hạn số lượng tác vụ ghi/giây (IOPS). |

### 🌐 Mạng (Network)
* **Lưu ý:** Docker Engine phiên bản Community (CE) mặc định **không** có flag `--network-add-limit`. 
* Để giới hạn băng thông mạng trong Docker, bạn thường phải dùng công cụ `tc` (Traffic Control) của Linux bên trong container hoặc sử dụng các Driver Network nâng cao.

---

## 🛠️ Cách kiểm tra thông số Cgroups thực tế
Bạn có thể xác nhận Docker đã thiết lập các giới hạn này vào Kernel bằng cách truy cập hệ thống file ảo `/sys/fs/cgroup/`:

```bash
# Lấy ID đầy đủ của container
export CID=$(docker ps -q --no-trunc -l)

# Kiểm tra giới hạn RAM (đơn vị: Bytes)
cat /sys/fs/cgroup/memory/docker/$CID/memory.limit_in_bytes

# Kiểm tra nhân CPU được cấp phát
cat /sys/fs/cgroup/cpuset/docker/$CID/cpuset.cpus
```
