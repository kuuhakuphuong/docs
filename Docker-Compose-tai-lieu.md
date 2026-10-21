# Docker Compose: tài liệu tổng quan về mô hình, kiến trúc, đối tượng, mạng, workflow, câu lệnh, và gỡ lỗi

> **Viết cho:** người muốn định nghĩa và chạy ứng dụng nhiều container bằng một file YAML, từ máy phát triển đến CI và môi trường thử nghiệm. Cấu trúc tương tự các tài liệu Docker, Swarm và Kubernetes để dễ đối chiếu.
> **Cách đọc:** Phần A đến C là mô hình và kiến trúc. Phần D đến G là mạng, cấu hình, phụ thuộc khởi động, và nhiều môi trường. Phần I là tham chiếu lệnh. Phần J là gỡ lỗi. Các file YAML, lệnh và Dockerfile có chú thích từng dòng bên cạnh.
> **Phiên bản tham chiếu:** Compose v2 (lệnh `docker compose`, là plugin của Docker CLI). Trang tham chiếu CLI không ghi số phiên bản cụ thể. Compose Specification là định dạng file mới nhất và được khuyến nghị. Đã được triển khai trong Docker Compose CLI từ 1.27.0.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với docs.docker.com; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các file compose và lệnh chưa được chạy thử trong lần này. Nên thử trên máy có Docker trước khi dùng cho dự án thật.

---

## Mục lục

- **Phần A. Compose là gì và giải quyết vấn đề gì**
  - 1. Từ `docker run` nhiều dòng đến một file
  - 2. Compose làm được gì và không làm được gì
  - 3. Compose v2 và v1
- **Phần B. Kiến trúc và cách hoạt động**
  - 4. Thành phần: CLI, Compose Specification, Docker Engine
  - 5. Project, service, container
  - 6. Quy trình `docker compose up`
- **Phần C. Các đối tượng trong file**
  - 7. Cấu trúc file và tên file
  - 8. Service
  - 9. Network
  - 10. Volume
  - 11. Config và secret
  - 12. Profile
- **Phần D. Mạng và kết nối**
  - 13. Mạng mặc định của project
  - 14. DNS theo tên service
  - 15. Publish cổng và alias
- **Phần E. Cấu hình và biến môi trường**
  - 16. Biến môi trường, `.env`, `env_file`
  - 17. Interpolation
  - 18. Bí mật
- **Phần F. Thứ tự khởi động và sức khỏe**
  - 19. `depends_on` và điều kiện
  - 20. Health check
  - 21. Chính sách khởi động lại
- **Phần G. Nhiều môi trường**
  - 22. File override
  - 23. Profile
  - 24. `include` và `extends`
  - 25. Chạy với `-f`, `-p`, `--env-file`
- **Phần H. Workflow**
  - 26. Workflow phát triển cục bộ
  - 27. Watch: tự đồng bộ khi sửa mã
  - 28. Kiểm thử trong Compose
  - 29. Tích hợp CI
  - 30. Compose trong môi trường sản xuất
- **Phần I. Câu lệnh**
  - 31. Vòng đời dịch vụ
  - 32. Xem trạng thái và log
  - 33. Chạy lệnh
  - 34. Build và image
  - 35. Cấu hình và kiểm tra
  - 36. Dọn dẹp
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 37. Quy trình gỡ lỗi
  - 38. Lỗi cú pháp và cấu hình
  - 39. Dịch vụ không khởi động
  - 40. Dịch vụ không gọi được nhau
  - 41. Volume và quyền
  - 42. Build lỗi hoặc cũ
  - 43. Biến môi trường không đúng
  - 44. Cổng bị chiếm
- **Phần K. Vận hành**
  - 45. Log, tài nguyên, dọn dẹp
  - 46. Sao lưu volume
- **Phần L. So sánh với Swarm và Kubernetes**
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. COMPOSE LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Từ `docker run` nhiều dòng đến một file

Ứng dụng thực tế thường có nhiều thành phần: web, API, cơ sở dữ liệu, cache. Chạy bằng `docker run` từng cái, phải nhớ cổng, mạng, biến môi trường, volume, thứ tự khởi động. Dễ sai và khó tái lập.

**Docker Compose** cho phép mô tả toàn bộ ứng dụng trong một file YAML, rồi tạo và chạy mọi thành phần bằng một lệnh. [Đã kiểm chứng: trang Compose nói có thể quản lý dịch vụ, mạng, volume trong một file và khởi động bằng một lệnh]

## 2. Compose làm được gì và không làm được gì

**Làm được:**
- Định nghĩa dịch vụ, mạng, volume, config, secret trong một file.
- Chạy cùng một cấu hình trên máy phát triển, CI, và môi trường thử nghiệm. [Đã kiểm chứng: trang nói Compose dùng được cho phát triển, staging, testing, CI]
- Build image từ Dockerfile và chạy ngay.
- Chạy lệnh một lần (`run`), xem log, vào shell (`exec`).

**Hạn chế:**
- Chủ yếu chạy trên **một máy**. Muốn nhiều máy, dùng Swarm hoặc Kubernetes. [Suy luận]
- Không tự phục hồi và co giãn như cluster orchestrator. [Suy luận]
- Không thay thế quản lý bí mật và hạ tầng chuyên nghiệp ở quy mô lớn. [Suy luận]

## 3. Compose v2 và v1

| Tiêu chí | Compose v2 | Compose v1 (cũ) |
|---|---|---|
| Lệnh | `docker compose` (plugin của Docker CLI) | `docker-compose` (chương trình riêng) |
| Trạng thái | Phiên bản hiện tại | Đã ngừng |
| Định dạng file | Compose Specification | Định dạng cũ hơn (2.x, 3.x) |

[Đã kiểm chứng: định dạng 2.x và 3.x đã được gộp vào Compose Specification, triển khai từ Compose CLI 1.27.0; lệnh `docker compose` là Compose v2]

**Lưu ý:** nếu bạn đọc ví dụ cũ dùng `docker-compose`, thay bằng `docker compose`. [Suy luận]

---

# PHẦN B. KIẾN TRÚC VÀ CÁCH HOẠT ĐỘNG

## 4. Thành phần: CLI, Compose Specification, Docker Engine

```
┌──────────────────────┐
│ File compose.yaml    │   Compose Specification: mô tả mong muốn
└──────────┬───────────┘
           │ đọc, tổng hợp, kiểm tra
┌──────────▼───────────┐
│ docker compose (CLI) │   plugin Docker CLI, dịch file thành các lệnh
└──────────┬───────────┘
           │ Docker Engine API
┌──────────▼───────────┐
│ dockerd              │   tạo network, volume, container, image
└──────────────────────┘
```

- **Compose Specification:** định dạng file chuẩn, mô tả services, networks, volumes, configs, secrets. Đây là định dạng mới nhất được khuyến nghị. [Đã kiểm chứng]
- **CLI:** đọc file, tính toán cấu hình cuối cùng, rồi gọi Docker Engine. Compose không tự chạy container; Engine làm việc đó. [Suy luận từ mô hình Docker client-server]

## 5. Project, service, container

- **Project:** một tập tài nguyên được quản lý cùng nhau, thường là một ứng dụng. Tên project mặc định lấy từ tên thư mục chứa file. [Trí nhớ]
- **Service:** định nghĩa một thành phần (ví dụ `web`). Một service có thể có nhiều container (khi scale). [Trí nhớ]
- **Container:** instance đang chạy của service, tên thường dạng `<project>-<service>-<số>`. [Trí nhớ]

```sh
docker compose -p demo ps                   # -p đặt tên project (mặc định là tên thư mục)
```

## 6. Quy trình `docker compose up`

```
1. Đọc file compose và file override (nếu có), kết hợp biến môi trường
2. Tạo mạng và volume của project (nếu chưa có)
3. Build image cho service có khóa build (nếu cần)
4. Kéo image cho service có khóa image (nếu chưa có)
5. Khởi động service theo thứ tự phụ thuộc (depends_on)
6. Chờ điều kiện (ví dụ healthy) nếu có
7. Báo trạng thái
```

[Suy luận từ cấu trúc Compose; thứ tự chi tiết cần đối chiếu theo phiên bản]

---

# PHẦN C. CÁC ĐỐI TƯỢNG TRONG FILE

## 7. Cấu trúc file và tên file

**Tên file ưu tiên:** `compose.yaml` (khuyến nghị). Các tên khác được hỗ trợ: `compose.yml`, `docker-compose.yaml`, `docker-compose.yml`. [Trí nhớ]

**Cấu trúc cơ bản:**

```yaml
name: shop                          # tên project (tùy chọn, ghi đè tên thư mục)
services:                           # khai báo các dịch vụ
  web: ...                          # dịch vụ web
  db: ...                           # dịch vụ db
networks:                           # khai báo mạng (tùy chọn)
  ...
volumes:                            # khai báo volume có tên (tùy chọn)
  ...
configs:                            # khai báo config (tùy chọn)
  ...
secrets:                            # khai báo secret (tùy chọn)
  ...
```

**Lưu ý:** khóa `version` ở đầu file không còn cần thiết với Compose Specification. Có thể gặp trong tài liệu cũ. [Trí nhớ]

## 8. Service

Ví dụ đầy đủ, mỗi dòng có chú thích:

```yaml
services:
  web:                              # tên service, cũng là tên DNS trong mạng project
    build:                          # build từ Dockerfile thay vì kéo image
      context: ./web                # thư mục ngữ cảnh build
      dockerfile: Dockerfile        # tên Dockerfile trong ngữ cảnh
    image: shop-web:dev             # gắn tên cho image vừa build
    ports:
      - "8000:8000"                 # publish cổng máy 8000 → container 8000
    environment:                    # biến môi trường
      DB_HOST: db                   # gọi service db bằng tên
      MUC_LOG: info
    env_file:
      - ./web.env                   # đọc biến từ file (không commit file này)
    volumes:
      - ./web/src:/app/src          # bind mount: mã nguồn trên máy vào container
      - web-cache:/app/.cache       # volume có tên
    depends_on:
      db:
        condition: service_healthy  # chờ db healthy
    restart: unless-stopped         # khởi động lại trừ khi dừng thủ công
    networks:
      - backend                     # gắn vào mạng backend
    command: ["python", "app.py"]   # ghi đè CMD của image
    working_dir: /app               # thư mục làm việc

  db:
    image: postgres:16              # image có sẵn từ registry
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password   # đọc mật khẩu từ file (image hỗ trợ _FILE)
    secrets:
      - db_password                 # gắn secret vào container
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]   # lệnh kiểm tra sức khỏe
      interval: 10s                 # kiểm tra mỗi 10 giây
      timeout: 3s                   # quá 3 giây là thất bại
      retries: 5                    # thử 5 lần
    networks:
      - backend
```

| Khóa | Ý nghĩa |
|---|---|
| `image` | Image có sẵn hoặc tên gắn cho image build |
| `build` | Ngữ cảnh và Dockerfile để build |
| `ports` | Publish cổng |
| `expose` | Ghi chú cổng nội bộ, không publish ra máy |
| `environment` | Biến môi trường |
| `env_file` | Biến đọc từ file |
| `volumes` | Gắn volume hoặc bind mount |
| `depends_on` | Thứ tự và điều kiện khởi động |
| `healthcheck` | Kiểm tra sức khỏe |
| `restart` | Chính sách khởi động lại |
| `networks` | Mạng của service |
| `command`, `entrypoint` | Ghi đè CMD và ENTRYPOINT |
| `secrets`, `configs` | Gắn bí mật và cấu hình |
| `profiles` | Nhóm service chỉ chạy khi bật profile (mục 23) |
| `deploy` | Khóa chủ yếu cho Swarm; Compose dùng một số phần như `resources` |

[Tên khóa và ý nghĩa là kiến thức nền của Compose Specification; cần đối chiếu từng khóa trong tài liệu tham chiếu]

## 9. Network

```yaml
networks:
  frontend:                         # mạng cho lớp giao diện
  backend:                          # mạng nội bộ cho cơ sở dữ liệu
    internal: true                  # không cho truy cập ra ngoài máy (không có đường ra internet) [Trí nhớ]
```

Service chỉ gắn vào mạng cần thiết, để giới hạn lưu lượng giữa các thành phần. [Suy luận]

## 10. Volume

```yaml
volumes:
  db-data:                          # volume có tên, Docker quản lý
  web-cache:                        # volume khác
```

Volume có tên tồn tại sau `docker compose down` (trừ khi dùng `-v`). [Trí nhớ]

## 11. Config và secret

```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt   # lấy nội dung từ file cục bộ (không commit)

configs:
  nginx_conf:
    file: ./nginx.conf                # cấu hình không nhạy cảm
```

Trong container, secret nằm tại `/run/secrets/<tên>`. [Trí nhớ]

## 12. Profile

Profile cho phép nhóm service chỉ chạy khi được yêu cầu, ví dụ công cụ gỡ lỗi hay công cụ quản trị:

```yaml
services:
  adminer:
    image: adminer
    profiles: ["debug"]             # chỉ chạy khi có profile debug
```

```sh
docker compose --profile debug up -d    # bật profile debug
```

---

# PHẦN D. MẠNG VÀ KẾT NỐI

## 13. Mạng mặc định của project

Compose tạo một mạng bridge riêng cho project (nếu không khai báo `networks`). Mọi service trong project gắn vào mạng này và gọi nhau được. [Trí nhớ]

Tên mạng thường dạng `<project>_default`.

## 14. DNS theo tên service

Trong mạng của project, mỗi service có tên DNS là tên khóa service. Trong ví dụ mục 8, `web` gọi `db:5432`. [Trí nhớ]

```sh
docker compose exec web getent hosts db       # kiểm tra phân giải tên db từ bên trong web
```

**Lưu ý:** dùng cổng nội bộ của container (`5432`), không dùng cổng publish ra máy (`15432`) khi các service gọi nhau. [Suy luận]

## 15. Publish cổng và alias

```yaml
services:
  web:
    ports:
      - "127.0.0.1:8000:8000"       # chỉ nghe trên localhost của máy (an toàn hơn)
    networks:
      backend:
        aliases:
          - api.local               # tên phụ trong mạng backend
```

**Lưu ý bảo mật:** `"5432:5432"` mở cổng ra mọi giao diện của máy. Nên bind `127.0.0.1` khi chỉ cần truy cập từ máy. [Suy luận]

---

# PHẦN E. CẤU HÌNH VÀ BIẾN MÔI TRƯỜNG

## 16. Biến môi trường, `.env`, `env_file`

Có ba nguồn biến:
- **`environment:`** trong file compose (ghi trực tiếp).
- **`env_file:`** đọc từ file chỉ định cho service.
- **Tệp `.env`** trong thư mục project: dùng để thay thế biến trong chính file compose (interpolation), không tự đưa vào container. [Trí nhớ]

```
# .env (cùng thư mục với compose.yaml)
TAG_PHIEN_BAN=1.4.2
CONG_WEB=8000
```

```yaml
services:
  web:
    image: shop-web:${TAG_PHIEN_BAN}     # thay bằng 1.4.2 khi đọc file
    ports:
      - "${CONG_WEB}:8000"               # thay bằng 8000
```

**Thứ tự ưu tiên** (từ cao đến thấp, theo kiến thức nền): biến trong shell, `--env-file` chỉ định, `.env` mặc định, giá trị mặc định trong file. [Trí nhớ]

## 17. Interpolation

Cú pháp biến trong file: [Trí nhớ]

| Cú pháp | Ý nghĩa |
|---|---|
| `${VAR}` | Giá trị của VAR |
| `${VAR:-mac-dinh}` | Dùng mặc định nếu VAR rỗng hoặc chưa đặt |
| `${VAR-mac-dinh}` | Dùng mặc định nếu VAR chưa đặt |
| `${VAR:?thong-bao}` | Báo lỗi và dừng nếu VAR rỗng |
| `$$` | Ký tự `$` thật |

```yaml
environment:
  MUC_LOG: ${MUC_LOG:-info}              # mặc định info
  API_KEY: ${API_KEY:?phải đặt API_KEY}  # bắt buộc phải có
```

## 18. Bí mật

- **Không** đặt mật khẩu trong `environment` hoặc `.env` nếu file được commit. [Suy luận]
- Dùng `secrets:` với file không commit, hoặc biến môi trường của CI.
- Nhiều image chính thức (như Postgres) hỗ trợ biến `_FILE` để đọc mật khẩu từ tệp. [Trí nhớ]
- Thêm `.env`, `secrets/` vào `.gitignore`. [Suy luận]

---

# PHẦN F. THỨ TỰ KHỞI ĐỘNG VÀ SỨC KHỎE

## 19. `depends_on` và điều kiện

`depends_on` chỉ định thứ tự khởi động, và có thể chờ một điều kiện: [Trí nhớ]

| Điều kiện | Ý nghĩa |
|---|---|
| `service_started` | Chờ container của dịch vụ phụ thuộc đã khởi động (mặc định) |
| `service_healthy` | Chờ health check của dịch vụ phụ thuộc báo `healthy` |
| `service_completed_successfully` | Chờ dịch vụ phụ thuộc chạy xong và thoát mã 0 (dùng cho tác vụ khởi tạo) |

```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy           # chờ db khỏe
      migrate:
        condition: service_completed_successfully   # chờ migrate chạy xong
  migrate:
    image: shop-web:dev
    command: ["python", "manage.py", "migrate"]   # tác vụ một lần
    depends_on:
      db:
        condition: service_healthy
```

**Lưu ý:** `depends_on` đảm bảo thứ tự khởi động, không đảm bảo ứng dụng bên trong đã sẵn sàng nếu không có health check. [Suy luận]

## 20. Health check

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/healthz"]   # lệnh kiểm tra (image phải có curl)
  interval: 30s                # khoảng cách giữa các lần kiểm tra
  timeout: 3s                  # thời gian tối đa cho một lần kiểm tra
  retries: 3                   # số lần thất bại liên tiếp để báo unhealthy
  start_period: 20s            # thời gian khởi động không tính lỗi
```

```sh
docker compose ps                         # cột STATUS hiển thị healthy/unhealthy
```

## 21. Chính sách khởi động lại

| Giá trị | Ý nghĩa |
|---|---|
| `no` | Không tự khởi động (mặc định) |
| `always` | Luôn khởi động lại |
| `on-failure` | Chỉ khi thoát mã khác 0 |
| `unless-stopped` | Luôn, trừ khi dừng thủ công |

[Trí nhớ]

---

# PHẦN G. NHIỀU MÔI TRƯỜNG

## 22. File override

Compose đọc `compose.yaml` và `compose.override.yaml` (nếu có) và gộp. File override thường dùng cho máy phát triển: [Trí nhớ]

```yaml
# compose.override.yaml (tự động được đọc cùng compose.yaml)
services:
  web:
    volumes:
      - ./web/src:/app/src          # bind mount mã nguồn, chỉ dùng khi phát triển
    environment:
      DEBUG: "1"                    # bật chế độ gỡ lỗi
```

**Nhiều môi trường:** tạo file riêng, ví dụ `compose.prod.yaml`, và chạy với `-f` để chỉ định thứ tự. [Trí nhớ]

```sh
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

**Lưu ý:** khi dùng `-f`, Compose không tự đọc `compose.override.yaml`. Phải khai báo đầy đủ. [Trí nhớ]

## 23. Profile

Đã mô tả ở mục 12. Profile phù hợp để bật/tắt công cụ phụ như công cụ quản trị, mock, hoặc tác vụ kiểm thử, mà không cần file riêng. [Suy luận]

## 24. `include` và `extends`

- **`include`:** nhúng một file compose khác (ví dụ của một thành phần dùng chung). [Trí nhớ]
- **`extends`:** kế thừa cấu hình của một service từ file khác. [Trí nhớ]

```yaml
include:
  - ../common/compose.yaml          # nhúng file dùng chung từ thư mục khác
```

## 25. Chạy với `-f`, `-p`, `--env-file`

```sh
docker compose -f compose.yaml -f compose.dev.yaml up -d    # nhiều file theo thứ tự
docker compose -p shop-dev up -d                            # tên project riêng (tách môi trường)
docker compose --env-file ./env/staging.env up -d           # chọn file biến môi trường
```

**Lưu ý:** cùng một file trên hai project khác tên tạo hai bộ tài nguyên khác nhau, cách ly nhau. Đây là cách chạy nhiều bản song song trên một máy. [Suy luận]

---

# PHẦN H. WORKFLOW

## 26. Workflow phát triển cục bộ

```
1. Viết compose.yaml với service ứng dụng và phụ thuộc (db, cache)
2. Đặt biến trong .env, bí mật trong file không commit
3. Khởi động: docker compose up -d --build
4. Kiểm tra trạng thái và log
5. Chạy lệnh tiện ích: migrate, seed, test
6. Sửa mã; dùng watch để đồng bộ tự động (mục 27)
7. Kết thúc: docker compose down
```

```sh
docker compose up -d --build                # build lại image nếu cần, khởi động nền
docker compose ps                           # trạng thái
docker compose logs -f web                  # theo dõi log web
docker compose exec db psql -U postgres     # vào psql trong container db
docker compose down                         # dừng, xóa container và mạng (giữ volume)
```

## 27. Watch: tự đồng bộ khi sửa mã

`docker compose watch` theo dõi thay đổi trong ngữ cảnh build, rồi đồng bộ, build lại, hoặc khởi động lại container: [Đã kiểm chứng: trang tham chiếu CLI có lệnh watch với mô tả này]

```yaml
services:
  web:
    build: ./web
    develop:                          # khóa dành cho docker compose watch
      watch:
        - action: sync                # đồng bộ file vào container, không build lại
          path: ./web/src             # thư mục theo dõi trên máy
          target: /app/src            # nơi nhận trong container
        - action: rebuild             # build lại khi thay đổi file phụ thuộc
          path: ./web/requirements.txt
```

```sh
docker compose watch                  # chạy theo dõi (giữ terminal mở)
```

[Tên khóa `develop` và `watch` cần kiểm tra theo tài liệu tham chiếu hiện hành]

## 28. Kiểm thử trong Compose

```sh
docker compose run --rm web pytest              # chạy test trong service web, xóa container sau khi xong
docker compose run --rm --no-deps web pytest    # không khởi động dịch vụ phụ thuộc
docker compose up -d db && docker compose run --rm web pytest   # kiểm thử với cơ sở dữ liệu thật
```

**Mẫu cho CI:** dùng một file riêng để test (ví dụ `compose.test.yaml`), chạy trong project tên duy nhất, rồi `down -v` để dọn sạch. [Suy luận]

## 29. Tích hợp CI

```yaml
name: test
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker compose -p ci-${{ github.run_id }} -f compose.yaml -f compose.test.yaml up --build --abort-on-container-exit --exit-code-from tests
      - if: always()
        run: docker compose -p ci-${{ github.run_id }} down -v   # dọn sạch dù thành công hay lỗi
```

**Giải thích lệnh:** `--abort-on-container-exit` dừng khi bất kỳ container nào thoát; `--exit-code-from tests` trả mã thoát của service `tests`. [Trí nhớ]

## 30. Compose trong môi trường sản xuất

Compose chạy được trên một máy sản xuất, nhưng cần cân nhắc: [Suy luận]
- Không có tự phục hồi trên nhiều máy và không có co giãn tự động.
- Cần sao lưu volume, giám sát, và quy trình cập nhật có kiểm soát.
- Pin phiên bản image bằng tag cụ thể hoặc digest.
- Tách cấu hình môi trường bằng file override hoặc `--env-file`, và giữ bí mật ngoài repo.
- Khi cần nhiều máy hoặc độ sẵn sàng cao, cân nhắc Swarm hoặc Kubernetes.

---

# PHẦN I. CÂU LỆNH

Cú pháp chung: `docker compose [tùy chọn toàn cục] <lệnh> [cờ] [service]`. Tùy chọn toàn cục gồm `-f` (file), `-p` (project), `--env-file`, `--profile`.

## 31. Vòng đời dịch vụ

```sh
docker compose up                       # tạo và khởi động, hiển thị log trực tiếp
docker compose up -d                    # khởi động nền
docker compose up -d --build            # build lại trước khi khởi động
docker compose up -d --force-recreate   # tạo lại container dù cấu hình không đổi
docker compose up -d web                # chỉ khởi động service web (và phụ thuộc)
docker compose start web                # bật service đã có, đã dừng
docker compose stop web                 # dừng service, giữ container
docker compose restart web              # khởi động lại
docker compose pause web                # tạm dừng tiến trình
docker compose unpause web              # tiếp tục
docker compose down                     # dừng và xóa container, mạng project
docker compose down -v                  # thêm xóa volume (mất dữ liệu)
docker compose down --rmi local         # thêm xóa image build cục bộ
docker compose rm -f                    # xóa container đã dừng
```

## 32. Xem trạng thái và log

```sh
docker compose ps                       # trạng thái service (chỉ project hiện tại)
docker compose ps -a                    # gồm cả container đã dừng
docker compose ls                       # liệt kê mọi project đang chạy
docker compose top                      # tiến trình trong các container
docker compose logs                     # log mọi service
docker compose logs -f web              # theo dõi log web
docker compose logs --tail 100 db       # 100 dòng cuối của db
docker compose events                   # sự kiện container
docker compose images                   # image đang dùng
docker compose stats                    # CPU, bộ nhớ (nếu có)
```

## 33. Chạy lệnh

```sh
docker compose exec web sh                        # shell trong container đang chạy
docker compose exec web python manage.py shell    # chạy lệnh
docker compose exec -T db pg_dump -U postgres app > backup.sql   # -T: không cấp TTY (dùng trong script)
docker compose run --rm web pytest                # container tạm, xóa sau khi chạy
docker compose run --rm --no-deps web sh          # không khởi động dịch vụ phụ thuộc
docker compose run --rm -e DEBUG=1 web pytest     # thêm biến môi trường cho lần chạy này
docker compose cp web:/app/log.txt ./log.txt      # sao chép từ container ra máy
docker compose cp ./config.txt web:/app/          # sao chép vào container
```

**Khác biệt `exec` và `run`:** `exec` chạy trong container đang có sẵn; `run` tạo container mới cho lần chạy đó. [Trí nhớ]

## 34. Build và image

```sh
docker compose build                        # build mọi service có khóa build
docker compose build web                    # chỉ web
docker compose build --no-cache web         # bỏ cache
docker compose build --pull web             # kéo image nền mới nhất trước khi build
docker compose pull                         # kéo image của các service
docker compose push                         # đẩy image đã build (nếu có khóa image)
```

## 35. Cấu hình và kiểm tra

```sh
docker compose config                       # in cấu hình đã tổng hợp và thay biến (kiểm tra cú pháp)
docker compose config --services            # chỉ tên service
docker compose config --volumes             # chỉ tên volume
docker compose config -q                    # chỉ kiểm tra, không in (mã thoát cho CI)
docker compose version                      # phiên bản Compose
```

**Gợi ý:** chạy `docker compose config` trước khi `up`, để thấy biến đã được thay và file override đã gộp đúng chưa. [Đã kiểm chứng: lệnh config render file ở dạng chuẩn]

## 36. Dọn dẹp

```sh
docker compose down                         # dọn tài nguyên của project
docker compose down -v --remove-orphans     # xóa volume và container mồ côi (mất dữ liệu)
docker system df                            # dung lượng (lệnh của Docker)
docker image prune                          # xóa image treo (lệnh của Docker)
```

**Cảnh báo:** `down -v` xóa volume có tên của project, nên dữ liệu cơ sở dữ liệu sẽ mất. Sao lưu trước. [Suy luận]

---

# PHẦN J. GỠ LỖI

## 37. Quy trình gỡ lỗi

```
1. Cú pháp file đúng không?         docker compose config
2. Biến và override đã gộp đúng?    docker compose config (xem kết quả đã tổng hợp)
3. Service nào lỗi?                 docker compose ps -a
4. Log nói gì?                      docker compose logs <service>
5. Container có chạy đúng lệnh?     docker compose exec / run để thử
6. Mạng và cổng?                    docker compose ps (cột PORTS), docker network ls
7. Volume và quyền?                 docker volume ls, docker compose exec <s> ls -la /đường-dẫn
```

## 38. Lỗi cú pháp và cấu hình

| Thông báo | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| `yaml: line N: ...` | Thụt lề sai, thiếu dấu hai chấm | Kiểm tra thụt lề bằng khoảng trắng, không dùng tab |
| `services.X additional properties not allowed` | Sai tên khóa | Đối chiếu tài liệu, dùng `docker compose config` |
| `invalid interpolation format` | Sai cú pháp `${...}` | Kiểm tra ngoặc và dấu `$` |
| `required variable X is missing` | Biến bắt buộc (`:?`) chưa đặt | Đặt trong `.env` hoặc shell |
| `service "X" depends on undefined service` | `depends_on` tham chiếu service không có | Sửa tên |

```sh
docker compose config -q                    # mã thoát khác 0 nếu cấu hình sai
```

## 39. Dịch vụ không khởi động

```sh
docker compose ps -a                        # trạng thái và mã thoát
docker compose logs --tail 100 <service>    # lỗi khởi động
docker compose run --rm --entrypoint sh <service>   # vào shell thử nghiệm
```

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| Thoát ngay với mã 0 | Lệnh chạy xong, không giữ tiến trình | Sửa `command` cho tiến trình chạy liên tục |
| Thoát với mã 1 | Lỗi ứng dụng, thiếu cấu hình | Đọc log |
| Không khởi động vì phụ thuộc | Điều kiện `service_healthy` không đạt | Kiểm tra health check của phụ thuộc |
| Lỗi `port is already allocated` | Cổng đã bị dùng | Đổi cổng publish |
| Lỗi khi build | Dockerfile hoặc ngữ cảnh sai | Xem mục 42 |

## 40. Dịch vụ không gọi được nhau

- Kiểm tra cả hai service có cùng mạng không: `docker compose config` (mục `networks`).
- Kiểm tra tên gọi: dùng tên service và cổng nội bộ, không dùng cổng publish.
- Kiểm tra ứng dụng nghe `0.0.0.0`, không phải `127.0.0.1`. [Suy luận]

```sh
docker compose exec web getent hosts db    # tên db phân giải được không
docker compose exec web sh -c 'nc -vz db 5432 || echo "không kết nối"'   # nếu image có nc [Trí nhớ]
```

## 41. Volume và quyền

```sh
docker compose config --volumes             # volume khai báo
docker volume ls                            # volume thật trong máy
docker compose exec <service> id            # UID đang chạy
docker compose exec <service> ls -la /data  # quyền thư mục
```

- Bind mount: đường dẫn trên máy phải tồn tại; quyền file trên máy quyết định quyền trong container. [Trí nhớ]
- Volume có tên lưu dữ liệu sau `down` (không có `-v`). Dữ liệu cũ có thể làm lỗi khởi động khi đổi phiên bản cơ sở dữ liệu. [Suy luận]

## 42. Build lỗi hoặc cũ

```sh
docker compose build --no-cache --progress=plain web    # build lại đầy đủ, xem chi tiết
docker compose up -d --build --force-recreate web       # build và tạo lại container
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| Sửa Dockerfile nhưng không đổi | Container cũ vẫn chạy | `up -d --build --force-recreate` |
| `COPY failed` | File không có trong ngữ cảnh, hoặc bị `.dockerignore` loại | Kiểm tra `context` và `.dockerignore` |
| Build chậm | Thứ tự chỉ thị không tối ưu cache | Xem tài liệu Dockerfile |
| Image cũ dù đã kéo | Không dùng `--pull` | `docker compose build --pull` hoặc `pull` |

## 43. Biến môi trường không đúng

```sh
docker compose config | grep -i <tên-biến>      # xem giá trị sau khi thay thế
docker compose exec web env | sort             # biến thật trong container đang chạy
```

**Thứ tự ưu tiên** và cách thay thế (mục 16–17). Lỗi thường gặp: `.env` không nằm cùng thư mục với compose file; file override ghi đè giá trị; biến trong shell ghi đè `.env`. [Trí nhớ]

## 44. Cổng bị chiếm

```sh
docker compose ps                           # cổng đã publish của project
docker ps --format "table {{.Names}}\t{{.Ports}}"   # mọi container trên máy
```

**Hướng xử lý:** đổi cổng bên trái (cổng máy), giữ cổng bên phải (cổng container). Ví dụ `"8001:8000"`. Hoặc dừng project đang chiếm cổng. [Suy luận]

---

# PHẦN K. VẬN HÀNH

## 45. Log, tài nguyên, dọn dẹp

- Đặt giới hạn log cho từng service để tránh đầy đĩa:

```yaml
services:
  web:
    logging:
      driver: json-file
      options:
        max-size: "10m"            # mỗi tệp log tối đa 10 MB
        max-file: "3"              # giữ 3 tệp
```

- Giới hạn tài nguyên: khóa `deploy.resources` cũng dùng được trong Compose cho một số trường hợp, hoặc dùng `mem_limit` và `cpus` tùy phiên bản. [Trí nhớ; cần kiểm tra]
- Dọn dẹp định kỳ: `docker system df` rồi `prune` có kiểm soát. [Suy luận]

## 46. Sao lưu volume

```sh
# Sao lưu volume của project
docker run --rm -v shop_db-data:/data -v "$(pwd):/backup" busybox \
  tar czf /backup/db-data.tar.gz -C /data .       # tên volume thường là <project>_<tên>

# Khôi phục
docker run --rm -v shop_db-data:/data -v "$(pwd):/backup" busybox \
  tar xzf /backup/db-data.tar.gz -C /data
```

**Lưu ý:** tên volume thật có tiền tố project (ví dụ `shop_db-data`). Kiểm tra bằng `docker volume ls`. Với cơ sở dữ liệu, nên dùng công cụ dump của chính cơ sở dữ liệu. [Suy luận]

---

# PHẦN L. SO SÁNH VỚI SWARM VÀ KUBERNETES

| Tiêu chí | Docker Compose | Docker Swarm | Kubernetes |
|---|---|---|---|
| Phạm vi | Một máy | Nhiều máy | Nhiều máy |
| Khai báo | compose.yaml | Stack file (Compose mở rộng) | Manifest YAML |
| Tự phục hồi | Hạn chế (`restart`) | Có | Có |
| Co giãn | Thủ công (`--scale` tùy phiên bản) | Thủ công (`scale`) | HPA tự động |
| Mạng | Bridge theo project | Overlay | CNI, Service |
| Mục đích chính | Phát triển, CI, môi trường nhỏ | Nhóm nhỏ, đơn giản | Quy mô lớn |
| Độ phức tạp | Thấp | Trung bình | Cao |

[Suy luận và kiến thức nền; so sánh khái quát]

**Đường đi thường gặp:** phát triển với Compose → chuyển sang Swarm hoặc Kubernetes khi cần nhiều máy. Stack file của Swarm dựa trên cú pháp Compose, nên một phần cấu hình có thể dùng lại. [Suy luận]

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Compose Specification | Định dạng file compose chuẩn hiện tại |
| Project | Tập tài nguyên của một ứng dụng, có tên riêng |
| Service | Định nghĩa một thành phần của ứng dụng |
| Container | Instance đang chạy của service |
| Interpolation | Thay thế biến `${...}` trong file |
| `.env` | File biến mặc định cạnh compose file, dùng để thay thế |
| `env_file` | Biến đọc vào container |
| Override | File bổ sung hoặc ghi đè cấu hình |
| Profile | Nhóm service bật theo yêu cầu |
| `depends_on` | Thứ tự và điều kiện khởi động |
| Health check | Kiểm tra sức khỏe container |
| `exec` / `run` | Chạy lệnh trong container đang có / container mới |
| Watch | Tự đồng bộ hoặc build lại khi mã đổi |
| Bind mount | Gắn thư mục từ máy vào container |
| Volume có tên | Volume do Docker quản lý, bền vững |
| Secret / Config | Bí mật / cấu hình gắn vào container |
| `include` / `extends` | Nhúng file / kế thừa service |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **Docker Compose (tổng quan)**: https://docs.docker.com/compose/ [đã đọc. Ghi nhận: quản lý dịch vụ, mạng, volume trong một file; khởi động bằng một lệnh; dùng được cho phát triển, staging, test, CI]
- **Compose Specification (tham chiếu file)**: https://docs.docker.com/reference/compose-file/ [đã đọc một phần. Ghi nhận: đây là định dạng mới nhất được khuyến nghị; gộp định dạng 2.x và 3.x; triển khai từ Compose CLI 1.27.0; trang không đầy đủ chi tiết khóa]
- **Tham chiếu lệnh `docker compose`**: https://docs.docker.com/reference/cli/docker/compose/ [đã đọc. Ghi nhận: danh sách lệnh up, down, ps, logs, exec, run, build, pull, push, config, restart, stop, start, rm, top, watch và các lệnh khác; không có số phiên bản]

## N.2. Nguồn chưa đối chiếu

- Chi tiết từng khóa service và top-level trong Compose Specification (depends_on, healthcheck, deploy, develop/watch, logging, networks với internal).
- Quy tắc thứ tự ưu tiên của biến môi trường và file override.
- Tài liệu `include`, `extends`, `profiles`.
- Tài liệu lệnh `docker compose watch` và cấu hình `develop`.
- Tài liệu mục `deploy.resources` và `mem_limit`/`cpus` trong Compose.

## N.3. Điểm cần xác minh

1. **Tên file ưu tiên và thứ tự tìm kiếm** (`compose.yaml` và các tên khác): kiến thức nền.
2. **Khóa `version`** có còn bắt buộc hay không: kiến thức nền cho rằng không cần thiết.
3. **Cú pháp `develop.watch`** và các action `sync`, `rebuild`: cần kiểm tra tài liệu watch.
4. **Biến `_FILE`** trong image Postgres: kiến thức nền, cần đối chiếu tài liệu của image.
5. **Thứ tự ưu tiên biến môi trường**: đánh dấu là kiến thức nền.
6. **Tên volume có tiền tố project** (`<project>_<tên>`): thường đúng, nhưng cần kiểm tra theo phiên bản.
7. **`mem_limit`, `cpus`, `deploy.resources`** trong Compose: cần kiểm tra cách hoạt động ngoài Swarm.
8. **Các ví dụ YAML, shell và GitHub Actions** chưa được chạy thử. Nên kiểm tra trên máy trước khi dùng.

---

**Tóm tắt để tra nhanh:**
- **Compose** = mô tả nhiều container trong một `compose.yaml`, chạy bằng `docker compose up`.
- **Kiến trúc:** CLI đọc Compose Specification, tổng hợp cấu hình, rồi gọi Docker Engine.
- **Đối tượng:** project, service, container, network, volume, config, secret, profile.
- **Mạng:** mạng riêng cho project; service gọi nhau bằng tên, dùng cổng nội bộ.
- **Cấu hình:** `.env` để thay thế biến, `env_file` để truyền vào container, secret thay cho biến mật khẩu.
- **Thứ tự và sức khỏe:** `depends_on` với `service_healthy` hoặc `service_completed_successfully`, kèm health check.
- **Nhiều môi trường:** file override, `-f` nhiều file, `-p` để tách project, profile để bật công cụ phụ.
- **Workflow:** `up -d --build` → `logs` → `exec`/`run` để thử → `watch` khi phát triển → `down` khi xong.
- **Gỡ lỗi:** `compose config` trước tiên, rồi `ps -a` và `logs`. Dùng `exec`/`run` để thử trong container.
- **Cẩn trọng:** `down -v` xóa dữ liệu, không commit `.env` và secret, và Compose không thay thế cluster khi cần nhiều máy.
