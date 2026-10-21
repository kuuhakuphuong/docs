# Docker: tài liệu tổng quan về kiến trúc, đối tượng, kết nối, workflow, câu lệnh, và gỡ lỗi

> **Viết cho:** người mới học Docker đến người xây dựng và vận hành container. Cấu trúc tương tự tài liệu Kubernetes để dễ đối chiếu: mô hình, kiến trúc, đối tượng, mạng, lưu trữ, workflow, câu lệnh, gỡ lỗi.
> **Cách đọc:** Phần A đến C là mô hình và kiến trúc. Phần D đến F là mạng, lưu trữ, và Dockerfile. Phần I là tham chiếu lệnh. Phần J là gỡ lỗi. Các đoạn lệnh, Dockerfile, và YAML có chú thích từng dòng ngay bên cạnh.
> **Phiên bản tham chiếu:** trang release notes chính thức (tra cứu 2026-10-21) liệt kê Docker Engine 29.x, bản mới nhất là **29.9.2** (2026-10-20). Trang không đánh dấu bản nào là "stable" chính thức. Trang cũng ghi daemon yêu cầu API v1.44 trở lên, và hỗ trợ cgroup v1 đến ít nhất tháng 5 năm 2029.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với docs.docker.com; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các lệnh và Dockerfile chưa được chạy thử trong lần này. Nên thử trên máy có Docker trước khi dùng cho dự án thật.

---

## Mục lục

- **Phần A. Docker là gì và giải quyết vấn đề gì**
  - 1. Vấn đề "chạy trên máy tôi thì được"
  - 2. Container khác máy ảo như thế nào
  - 3. Phiên bản và chính sách
- **Phần B. Kiến trúc**
  - 4. Mô hình client-server
  - 5. Daemon, containerd, runc
  - 6. Registry
  - 7. BuildKit và buildx
  - 8. Docker Desktop và Docker Compose
  - 9. Luồng một lệnh `docker run` đi qua các thành phần
- **Phần C. Các đối tượng cốt lõi**
  - 10. Image và layer
  - 11. Container và vòng đời
  - 12. Tag, digest, và đặt tên image
  - 13. Dockerfile: mô hình xây dựng
- **Phần D. Mạng**
  - 14. Các loại mạng
  - 15. Publish cổng và kết nối giữa container
  - 16. DNS trên mạng do người dùng tạo
- **Phần E. Lưu trữ**
  - 17. Volume, bind mount, tmpfs
  - 18. Quản lý volume
- **Phần F. Viết Dockerfile tốt**
  - 19. Các chỉ thị thường dùng
  - 20. Multi-stage build
  - 21. `.dockerignore` và cache layer
  - 22. Build với BuildKit
- **Phần G. Bảo mật**
  - 23. Chạy không phải root
  - 24. Hạn chế quyền và hệ tệp
  - 25. Bí mật (secret)
  - 26. Quét lỗ hổng và tin cậy image
  - 27. Rootless và cô lập
- **Phần H. Workflow**
  - 28. Workflow phát triển cục bộ
  - 29. Workflow build, tag, và đẩy image
  - 30. Docker Compose
  - 31. Tích hợp CI/CD
- **Phần I. Câu lệnh**
  - 32. Thông tin và hệ thống
  - 33. Image
  - 34. Container
  - 35. Volume
  - 36. Network
  - 37. Build và buildx
  - 38. Registry và đăng nhập
  - 39. Compose
  - 40. Dọn dẹp và dung lượng
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 41. Quy trình gỡ lỗi
  - 42. Lỗi build
  - 43. Lỗi container không khởi động hoặc thoát
  - 44. Lỗi mạng và cổng
  - 45. Lỗi volume và quyền
  - 46. Lỗi registry và kéo image
  - 47. Lỗi tài nguyên và hiệu năng
  - 48. Lỗi daemon và Docker Desktop
- **Phần K. Quan sát và vận hành**
  - 49. Log, stats, events
  - 50. Health check
  - 51. Cập nhật, sao lưu, và bảo trì
- **Phần L. Thực hành sản xuất**
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. DOCKER LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Vấn đề "chạy trên máy tôi thì được"

Một ứng dụng phụ thuộc vào: hệ điều hành, phiên bản ngôn ngữ, thư viện hệ thống, biến môi trường, cấu hình. Khi chuyển từ máy lập trình sang máy chủ, bất kỳ khác biệt nào cũng có thể làm ứng dụng lỗi. [Suy luận]

**Docker** đóng gói ứng dụng cùng toàn bộ môi trường cần thiết vào một **image**, và chạy image đó như một **container** cô lập. Cùng một image chạy giống nhau trên máy lập trình, máy CI, và máy chủ. [Suy luận từ mô hình image và container]

## 2. Container khác máy ảo như thế nào

| Tiêu chí | Container | Máy ảo (VM) |
|---|---|---|
| Nhân hệ điều hành | Dùng chung nhân của máy chủ | Có nhân riêng |
| Khởi động | Thường trong giây | Thường trong phút |
| Kích thước | Thường nhỏ, tính bằng MB | Thường lớn, tính bằng GB |
| Cô lập | Cô lập tiến trình, mạng, hệ tệp | Cô lập mạnh hơn ở cấp phần cứng ảo |
| Mật độ | Nhiều container trên một máy | Ít VM hơn trên cùng phần cứng |

[Suy luận và kiến thức nền; container dùng chung nhân Linux là đặc điểm cơ bản]

**Hệ quả:** container nhẹ và nhanh, nhưng cô lập yếu hơn VM. Với mã không tin cậy, cần các lớp bảo vệ bổ sung (mục 23–27). [Suy luận]

## 3. Phiên bản và chính sách

Theo trang release notes của Docker Engine: [Đã kiểm chứng]

- Bản mới nhất được liệt kê là **29.9.2** (2026-10-20). Trang có danh sách dày các bản vá trong dòng 29.x.
- Trang **không** đánh dấu bản nào là stable chính thức. Cần kiểm tra thêm trước khi chọn cho sản xuất.
- Daemon hiện yêu cầu **API version v1.44 trở lên**. Client cũ có thể không kết nối được.
- Hỗ trợ **cgroup v1** tiếp tục đến ít nhất tháng 5 năm 2029.
- containerd và BuildKit được phát hành cùng Engine, phiên bản đi kèm được ghi trong release notes (ví dụ 29.9.1 đi kèm containerd v2.4.2).

**Khuyến nghị:** ghim phiên bản Docker Engine và phiên bản image cơ sở trong môi trường sản xuất. [Suy luận]

---

# PHẦN B. KIẾN TRÚC

## 4. Mô hình client-server

Docker dùng kiến trúc client-server. Client gửi yêu cầu đến daemon, daemon xây dựng, chạy, và phân phối container. [Đã kiểm chứng]

```
┌─────────────┐   REST API (Unix socket hoặc TCP)   ┌────────────────┐
│ docker CLI  │ ──────────────────────────────────► │ dockerd        │
│ (client)    │                                      │ (Docker daemon)│
└─────────────┘                                      └───────┬────────┘
                                                             │
                 ┌───────────────────┬───────────────────────┼─────────────────┐
                 ▼                   ▼                       ▼                 ▼
          containerd            BuildKit             Registry (pull/push)   Network/Volume
          (quản lý vòng đời)   (build image)         (Docker Hub, riêng)    (quản lý)
                 │
                 ▼
               runc  (chạy container theo tiêu chuẩn OCI)
```

**Lưu ý:** sơ đồ đơn giản hóa. Vị trí chính xác của containerd và BuildKit có thể thay đổi theo cách cài đặt. [Suy luận]

## 5. Daemon, containerd, runc

| Thành phần | Vai trò | Ghi chú |
|---|---|---|
| **Docker CLI** (`docker`) | Giao diện dòng lệnh | Gửi lệnh tới dockerd qua REST API |
| **dockerd** | Daemon, quản lý image, container, volume, network | Lắng nghe API qua Unix socket hoặc mạng |
| **containerd** | Quản lý vòng đời container và image ở mức thấp | Được dockerd gọi [Trí nhớ] |
| **runc** | Chạy container theo đặc tả OCI | Tạo namespace và cgroup [Trí nhớ] |

[Đã kiểm chứng: trang tổng quan nói dockerd quản lý các đối tượng và gọi các daemon khác. Chi tiết containerd và runc là kiến thức nền]

**Socket mặc định** trên Linux: `/var/run/docker.sock`. [Trí nhớ]

**Cảnh báo bảo mật:** ai truy cập được socket Docker có quyền tương đương root trên máy chủ. Không mount socket vào container không tin cậy. [Trí nhớ]

## 6. Registry

**Registry** lưu và phân phối image. [Đã kiểm chứng]
- **Docker Hub** là registry công khai mặc định.
- Registry riêng (self-hosted hoặc dịch vụ đám mây) dùng cho mã nội bộ.
- `docker pull` tải image về, `docker push` đẩy lên, `docker run` tự kéo nếu thiếu image cục bộ.

## 7. BuildKit và buildx

**BuildKit** là engine build hiện đại, hỗ trợ build song song, cache hiệu quả, và nhiều đầu ra. [Trí nhớ]
**buildx** là plugin của Docker CLI để dùng BuildKit với nhiều tính năng nâng cao, như build đa nền tảng. [Đã kiểm chứng: danh sách lệnh có `buildx` và `builder`]

Release notes đã đọc có ghi phiên bản BuildKit đi kèm từng bản Engine (ví dụ v0.34.0 đi kèm 29.9.0). [Đã kiểm chứng]

## 8. Docker Desktop và Docker Compose

- **Docker Desktop:** ứng dụng cài đặt cho Mac, Windows, Linux. Gói daemon, client, Compose, và tiện ích khác, kể cả Kubernetes và Credential Helper. [Đã kiểm chứng]
- **Docker Compose:** công cụ để định nghĩa và chạy ứng dụng gồm nhiều container, bằng một tệp YAML. [Đã kiểm chứng]

## 9. Luồng một lệnh `docker run` đi qua các thành phần

```
1. Client gửi yêu cầu tạo container đến dockerd qua API
2. dockerd kiểm tra image cục bộ; nếu thiếu, kéo từ registry
3. dockerd tạo cấu hình: mạng, volume, biến môi trường, cổng
4. containerd nhận yêu cầu, chuẩn bị snapshot của hệ tệp từ các layer
5. runc tạo namespace và cgroup, khởi chạy tiến trình chính
6. Container chạy; dockerd theo dõi trạng thái
```

[Suy luận từ kiến trúc; các bước là minh họa]

---

# PHẦN C. CÁC ĐỐI TƯỢNG CỐT LÕI

## 10. Image và layer

**Image** là mẫu chỉ đọc chứa hệ tệp và cấu hình để tạo container. [Đã kiểm chứng]

**Layer:** mỗi chỉ thị trong Dockerfile tạo một layer. Layer là bất biến và được chia sẻ giữa các image. [Đã kiểm chứng: mô tả mỗi chỉ thị tạo layer]

```
Image ubuntu:24.04     ← layer 1 (base)
  + RUN apt install    ← layer 2
  + COPY app/          ← layer 3
  + CMD ["python"]    ← metadata (không thêm dữ liệu)
```

**Lợi ích:** layer được cache và chia sẻ, nên kéo và build nhanh hơn khi nội dung không đổi. [Suy luận]

```sh
docker image history <image>              # xem các layer và kích thước
```

## 11. Container và vòng đời

**Container** là một instance đang chạy (hoặc đã dừng) của image, có lớp ghi riêng ở trên cùng. [Đã kiểm chứng]

**Trạng thái thường gặp:** `created`, `running`, `paused`, `restarting`, `exited`, `dead`. [Trí nhớ]

```sh
docker create --name demo nginx:1.27    # tạo, chưa chạy
docker start demo                       # khởi động
docker stop demo                        # dừng nhẹ (SIGTERM, rồi SIGKILL sau timeout)
docker kill demo                        # gửi tín hiệu ngay (mặc định SIGKILL)
docker rm demo                          # xóa container đã dừng
```

**Lớp ghi riêng:** thay đổi trong container được ghi vào lớp này. Khi xóa container, các thay đổi biến mất, trừ khi dùng volume. [Đã kiểm chứng: trang tổng quan nói rõ dữ liệu mất khi xóa nếu không lưu bền vững]

## 12. Tag, digest, và đặt tên image

```
registry.example.com/team/web:1.4.2
│                      │    │     │
│                      │    │     └─ tag (nhãn phiên bản, có thể thay đổi)
│                      │    └─ tên repository
│                      └─ namespace
└─ registry (bỏ qua nghĩa là Docker Hub)
```

- **Tag:** nhãn dễ đọc, có thể được gán lại cho image khác. `latest` là tag mặc định nếu bỏ trống, nhưng không đảm bảo là bản mới nhất. [Trí nhớ]
- **Digest:** mã băm `sha256:...` xác định chính xác nội dung image, không đổi. [Trí nhớ]

```sh
docker pull nginx:1.27@sha256:<digest>   # ghim theo digest để tái lập chính xác
```

**Khuyến nghị:** dùng tag cụ thể (không dùng `latest`) và ghim digest cho image sản xuất. [Suy luận]

## 13. Dockerfile: mô hình xây dựng

**Dockerfile** là kịch bản mô tả cách build image. Mỗi chỉ thị thường tạo một layer.

```dockerfile
# syntax=docker/dockerfile:1          # chỉ định phiên bản cú pháp Dockerfile
FROM python:3.12-slim                  # image nền, có sẵn Python 3.12 bản nhỏ
WORKDIR /app                           # đặt thư mục làm việc, tạo nếu chưa có
COPY requirements.txt .                # sao chép file phụ thuộc trước (để tận dụng cache)
RUN pip install --no-cache-dir -r requirements.txt   # cài thư viện, không giữ cache pip
COPY . .                               # sao chép toàn bộ mã nguồn (đặt sau để cache tốt hơn)
EXPOSE 8000                            # ghi chú cổng ứng dụng lắng nghe (không mở cổng thật)
USER 10001                             # chạy bằng UID không phải root
CMD ["python", "app.py"]               # lệnh mặc định khi container khởi động
```

| Chỉ thị | Ý nghĩa |
|---|---|
| `FROM` | Image nền, luôn là dòng đầu có ý nghĩa |
| `WORKDIR` | Đặt thư mục hiện tại cho các lệnh sau |
| `COPY` / `ADD` | Sao chép từ ngữ cảnh build vào image. `ADD` có thêm tính năng giải nén và URL, nên ưu tiên `COPY` |
| `RUN` | Chạy lệnh trong lúc build, tạo layer |
| `ENV` | Biến môi trường |
| `ARG` | Biến chỉ có khi build |
| `EXPOSE` | Ghi chú cổng, không publish thật |
| `USER` | Người dùng chạy tiến trình |
| `ENTRYPOINT` | Lệnh cố định khi chạy container |
| `CMD` | Tham số mặc định, có thể ghi đè khi `docker run` |
| `HEALTHCHECK` | Lệnh kiểm tra sức khỏe container |
| `VOLUME` | Khai báo điểm gắn volume |

**Khác biệt ENTRYPOINT và CMD:** `ENTRYPOINT` là lệnh chính, `CMD` là tham số mặc định. Khi kết hợp, `CMD` là đối số của `ENTRYPOINT`. [Trí nhớ]

**Dạng exec và shell:** `CMD ["python", "app.py"]` (dạng mảng, không qua shell, nhận tín hiệu đúng) khác `CMD python app.py` (qua shell, tín hiệu có thể không đến tiến trình chính). [Trí nhớ]

---

# PHẦN D. MẠNG

## 14. Các loại mạng

| Driver | Đặc điểm | Dùng khi |
|---|---|---|
| **bridge** (mặc định) | Mạng ảo nội bộ trên máy chủ, NAT ra ngoài | Container chạy trên một máy |
| **host** | Dùng chung mạng với máy chủ, không cô lập | Cần hiệu năng mạng cao, hoặc cổng động [Trí nhớ] |
| **none** | Không có mạng | Container cô lập hoàn toàn [Trí nhớ] |
| **overlay** | Mạng trải qua nhiều máy (Swarm) | Ứng dụng phân tán [Trí nhớ] |
| **macvlan** | Container có địa chỉ MAC và IP riêng trên mạng vật lý | Tích hợp mạng sẵn có [Trí nhớ] |

```sh
docker network ls                                       # liệt kê mạng
docker network create app-net                           # tạo mạng bridge do người dùng tạo
docker run -d --name db --network app-net postgres:16   # chạy container trong mạng app-net
docker network inspect app-net                          # xem subnet và container kết nối
```

**Mạng do người dùng tạo (user-defined bridge)** có DNS nội bộ: container gọi nhau bằng tên, như `db`. Mạng bridge mặc định không có tính năng này. [Trí nhớ]

## 15. Publish cổng và kết nối giữa container

```sh
docker run -d -p 8080:80 nginx:1.27          # máy 8080 → container 80 (publish)
docker run -d -p 127.0.0.1:8080:80 nginx     # chỉ nghe trên localhost của máy
docker run -d -P nginx                       # publish mọi cổng EXPOSE ngẫu nhiên
docker port <container>                      # xem ánh xạ cổng
```

| Cờ | Ý nghĩa |
|---|---|
| `-p host:container` | Ánh xạ một cổng |
| `-p IP:host:container` | Chỉ nghe trên IP cụ thể (ví dụ 127.0.0.1) |
| `-P` | Publish mọi cổng đã khai báo bằng EXPOSE |

**Lưu ý bảo mật:** `-p 5432:5432` mặc định mở ra mọi giao diện mạng của máy. Chỉ publish khi cần, và giới hạn IP nếu có thể. [Suy luận]

**Kết nối giữa container:** trong cùng mạng do người dùng tạo, dùng tên container và cổng nội bộ, không cần publish. [Trí nhớ]

```sh
docker run -d --name api --network app-net mon-api:1.0   # gọi db bằng tên "db" và cổng 5432
```

## 16. DNS trên mạng do người dùng tạo

Trong mạng bridge do người dùng tạo, container phân giải được tên của container khác. Trong ví dụ trên, `api` kết nối tới `db:5432`. [Trí nhớ]

```sh
docker run --rm --network app-net busybox nslookup db    # kiểm tra phân giải tên
```

---

# PHẦN E. LƯU TRỮ

## 17. Volume, bind mount, tmpfs

| Loại | Nguồn | Vòng đời | Dùng cho |
|---|---|---|---|
| **Volume** | Docker quản lý | Tồn tại sau khi xóa container | Dữ liệu cơ sở dữ liệu, dữ liệu bền vững |
| **Bind mount** | Thư mục cụ thể trên máy chủ | Phụ thuộc máy chủ | Phát triển: mã nguồn đồng bộ với container |
| **tmpfs** | Bộ nhớ RAM | Mất khi dừng | Dữ liệu tạm nhạy cảm, cache |

```sh
docker volume create du-lieu-db                                   # tạo volume có tên
docker run -d -v du-lieu-db:/var/lib/postgresql/data postgres:16  # gắn volume vào thư mục dữ liệu
docker run -d -v "$(pwd)/src:/app/src" node:22 npm run dev        # bind mount: mã nguồn trên máy
docker run --rm --tmpfs /tmp:rw,size=64m alpine sh                # tmpfs 64 MB tại /tmp
```

**Lưu ý:** cú pháp `-v` và `--mount` khác nhau về độ rõ ràng. `--mount` tường minh hơn và khuyến nghị cho người mới. [Trí nhớ]

```sh
docker run -d --mount type=volume,source=du-lieu-db,target=/var/lib/postgresql/data postgres:16
docker run -d --mount type=bind,source="$(pwd)/src",target=/app/src node:22
```

## 18. Quản lý volume

```sh
docker volume ls                              # liệt kê volume
docker volume inspect du-lieu-db              # xem vị trí lưu trữ và thông tin
docker volume rm du-lieu-db                   # xóa volume (mất dữ liệu)
docker volume prune                           # xóa volume không dùng (thận trọng)
```

**Cảnh báo:** `docker volume prune` và `docker system prune --volumes` xóa dữ liệu không được container nào dùng. Kiểm tra trước khi chạy. [Suy luận]

---

# PHẦN F. VIẾT DOCKERFILE TỐT

## 19. Các chỉ thị thường dùng và nguyên tắc

Đã mô tả ở mục 13. Nguyên tắc: **thứ thay đổi ít đặt trước, thứ thay đổi nhiều đặt sau**, để cache hoạt động tốt. [Suy luận]

## 20. Multi-stage build

Multi-stage tách môi trường build khỏi image chạy, giảm kích thước và bề mặt tấn công:

```dockerfile
# Giai đoạn 1: build
FROM golang:1.23 AS build                 # đặt tên giai đoạn là build
WORKDIR /src                              # thư mục làm việc
COPY go.mod go.sum ./                     # sao chép file phụ thuộc
RUN go mod download                       # tải phụ thuộc (cache được)
COPY . .                                  # sao chép mã nguồn
RUN CGO_ENABLED=0 go build -o /out/app .  # build tệp nhị phân tĩnh

# Giai đoạn 2: image chạy nhỏ
FROM gcr.io/distroless/static:nonroot     # image tối giản, không có shell, chạy non-root
COPY --from=build /out/app /app           # chỉ sao chép tệp nhị phân từ giai đoạn build
USER nonroot                              # chạy bằng người dùng không phải root
ENTRYPOINT ["/app"]                       # lệnh chính
```

[Đã kiểm chứng: cú pháp multi-stage và `COPY --from` là chuẩn; tên image distroless cần kiểm tra]

**Lợi ích:** image chạy không chứa compiler, công cụ build, mã nguồn. [Suy luận]

## 21. `.dockerignore` và cache layer

`.dockerignore` loại bỏ file khỏi ngữ cảnh build, giảm thời gian gửi và tránh đưa bí mật vào image:

```
.git
node_modules
*.log
.env
```

**Cache layer:** Docker dùng lại layer nếu chỉ thị và đầu vào không đổi. Một thay đổi làm layer đó và mọi layer sau nó được build lại. [Trí nhớ]

## 22. Build với BuildKit

```sh
docker build -t web:1.4.2 .                               # build với tag
docker buildx build -t web:1.4.2 --load .                 # buildx, nạp image vào Docker
docker buildx build --platform linux/amd64,linux/arm64 -t registry.example.com/web:1.4.2 --push .   # đa nền tảng, đẩy trực tiếp
docker buildx build --secret id=npmrc,src=$HOME/.npmrc .  # truyền bí mật lúc build, không lưu trong layer [Trí nhớ]
```

**Lưu ý:** bí mật truyền bằng `ARG` hoặc `COPY` có thể lưu trong layer và bị lộ khi đẩy image. Dùng cơ chế secret của BuildKit. [Trí nhớ]

---

# PHẦN G. BẢO MẬT

## 23. Chạy không phải root

```dockerfile
RUN addgroup --system app && adduser --system --ingroup app app   # tạo người dùng (cú pháp tùy distro)
USER app                                                          # chuyển sang người dùng này
```

```sh
docker run --user 10001:10001 web:1.4.2       # chỉ định UID:GID khi chạy
```

[Cú pháp tạo người dùng phụ thuộc hệ điều hành của image nền]

## 24. Hạn chế quyền và hệ tệp

```sh
docker run --read-only --tmpfs /tmp web:1.4.2          # hệ tệp gốc chỉ đọc, /tmp là tmpfs
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE web:1.4.2   # bỏ mọi capability, chỉ thêm cần thiết
docker run --security-opt no-new-privileges web:1.4.2  # chặn leo quyền
docker run --memory=512m --cpus=1 web:1.4.2            # giới hạn bộ nhớ và CPU
docker run --pids-limit=200 web:1.4.2                  # giới hạn số tiến trình
```

**Nguyên tắc:** bỏ mọi quyền, rồi thêm lại đúng cái cần. [Suy luận]

## 25. Bí mật (secret)

- **Không** đặt mật khẩu trong Dockerfile, `ENV`, hoặc `ARG` (có thể lưu trong lịch sử image). [Trí nhớ]
- Truyền bí mật lúc chạy bằng biến môi trường từ file, hoặc secret của Compose/Swarm. [Trí nhớ]
- Kiểm tra lịch sử image bằng `docker history` để đảm bảo không có bí mật. [Trí nhớ]

```sh
docker history --no-trunc web:1.4.2    # xem đầy đủ lệnh trong từng layer
```

## 26. Quét lỗ hổng và tin cậy image

```sh
docker scout quickview web:1.4.2       # tổng quan lỗ hổng (Docker Scout) [Đã kiểm chứng: lệnh scout có trong danh sách]
docker scout cves web:1.4.2            # chi tiết CVE
```

Các công cụ khác như Trivy hoặc Grype cũng dùng được. [Trí nhớ]

**Nguyên tắc:** chỉ dùng image nền từ nguồn tin cậy, ghim digest, và quét định kỳ. [Suy luận]

## 27. Rootless và cô lập

- **Rootless mode:** daemon chạy bằng người dùng thường, giảm tác động khi bị khai thác. [Trí nhớ]
- **Không mount `/var/run/docker.sock`** vào container không tin cậy. [Trí nhớ]
- **Không dùng `--privileged`** trừ khi thật sự cần, vì cấp gần như mọi quyền của máy chủ. [Trí nhớ]

---

# PHẦN H. WORKFLOW

## 28. Workflow phát triển cục bộ

```
1. Viết mã và Dockerfile
2. Build image với tag rõ ràng
3. Chạy container, kiểm tra cổng và log
4. Gắn bind mount để sửa mã và xem thay đổi (phát triển)
5. Dùng Compose để chạy cả hệ thống (cơ sở dữ liệu, cache, API)
6. Kiểm thử trong container
7. Dọn dẹp tài nguyên không dùng
```

```sh
docker build -t api:dev .
docker run --rm -p 8000:8000 api:dev
docker logs -f <container>
docker system prune -f                    # dọn dẹp (thận trọng)
```

## 29. Workflow build, tag, và đẩy image

```sh
docker build -t registry.example.com/team/web:1.4.2 .        # build với tag đầy đủ
docker login registry.example.com                           # đăng nhập registry
docker push registry.example.com/team/web:1.4.2             # đẩy lên registry
docker image inspect --format '{{index .RepoDigests 0}}' registry.example.com/team/web:1.4.2   # lấy digest
docker logout registry.example.com                          # đăng xuất
```

**Quy tắc tag:** gắn cả tag phiên bản và, nếu cần, tag theo commit (ví dụ `1.4.2` và `sha-abc123`). Không dựa vào `latest`. [Suy luận]

## 30. Docker Compose

Compose định nghĩa nhiều dịch vụ trong một tệp `compose.yaml`: [Đã kiểm chứng: Compose là công cụ định nghĩa và chạy ứng dụng nhiều container]

```yaml
services:                                  # danh sách dịch vụ
  web:                                     # tên dịch vụ, cũng là tên DNS trong mạng Compose
    build: .                               # build từ Dockerfile trong thư mục hiện tại
    ports:
      - "8000:8000"                        # publish cổng máy 8000 → container 8000
    environment:
      DB_HOST: db                          # gọi dịch vụ db bằng tên
      DB_PASSWORD_FILE: /run/secrets/db_password   # đọc bí mật từ file
    depends_on:
      db:
        condition: service_healthy         # chờ db báo healthy trước khi khởi động
    secrets:
      - db_password
    restart: unless-stopped                # tự khởi động lại trừ khi dừng thủ công

  db:
    image: postgres:16                     # image có sẵn
    volumes:
      - db-data:/var/lib/postgresql/data   # volume bền vững
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]   # lệnh kiểm tra
      interval: 10s                        # mỗi 10 giây
      retries: 5                           # thử 5 lần

volumes:
  db-data:                                 # khai báo volume có tên

secrets:
  db_password:
    file: ./secrets/db_password.txt        # nguồn bí mật từ file cục bộ (không commit)
```

```sh
docker compose up -d                       # khởi động nền
docker compose ps                          # trạng thái dịch vụ
docker compose logs -f web                 # log dịch vụ web
docker compose exec web sh                 # mở shell trong dịch vụ
docker compose down                        # dừng và xóa container, mạng (giữ volume)
docker compose down -v                     # xóa cả volume (mất dữ liệu)
```

**Lưu ý:** `docker compose` (có khoảng trắng) là plugin Compose v2, khác với lệnh `docker-compose` cũ. [Trí nhớ]

## 31. Tích hợp CI/CD

```yaml
name: build
on: [push]
jobs:
  image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3          # cài buildx
      - uses: docker/login-action@v3                 # đăng nhập registry
        with:
          registry: registry.example.com
          username: ${{ secrets.REG_USER }}          # lấy từ secret của CI
          password: ${{ secrets.REG_PASS }}
      - uses: docker/build-push-action@v6            # build và đẩy
        with:
          push: true
          tags: registry.example.com/team/web:${{ github.sha }}   # tag theo commit
```

[Tên action và phiên bản cần kiểm tra theo tài liệu hiện hành]

**Pipeline điển hình:** lint Dockerfile, build, quét lỗ hổng, kiểm thử trong container, đẩy image, ghi lại digest để triển khai.

---

# PHẦN I. CÂU LỆNH

Cú pháp chung: `docker <nhóm> <lệnh> [cờ] [đối số]`. Nhiều lệnh có dạng rút gọn cũ (`docker ps`, `docker images`) và dạng nhóm mới (`docker container ls`).

## 32. Thông tin và hệ thống

```sh
docker version                          # phiên bản client và server
docker info                             # thông tin daemon: driver, số container, tài nguyên
docker system df                        # dung lượng đang dùng
docker system events                    # sự kiện daemon theo thời gian thực
docker context ls                       # danh sách context (máy đích)
docker context use <tên>                # chuyển context
```

## 33. Image

```sh
docker images                           # liệt kê image (tương đương docker image ls)
docker pull nginx:1.27                  # tải image
docker image inspect nginx:1.27         # chi tiết JSON: cấu hình, layer
docker image history nginx:1.27         # các layer và kích thước
docker tag web:1.4.2 registry.example.com/team/web:1.4.2   # gắn thêm tag
docker rmi web:1.4.2                    # xóa image (không xóa nếu container đang dùng)
docker image prune                      # xóa image treo (dangling)
docker image prune -a                   # xóa mọi image không được container nào dùng (thận trọng)
docker save -o web.tar web:1.4.2        # xuất image thành tệp tar
docker load -i web.tar                  # nhập image từ tar
docker search nginx                     # tìm trên registry
```

## 34. Container

```sh
docker run -d --name web -p 8080:80 nginx:1.27        # chạy nền, đặt tên, publish cổng
docker run --rm -it alpine sh                         # chạy tương tác, xóa khi thoát
docker run -e MUC_LOG=info -e CHE_DO=prod web:1.4.2   # truyền biến môi trường
docker run --env-file .env web:1.4.2                  # truyền biến từ file
docker run -v data:/app/data web:1.4.2                # gắn volume
docker run --network app-net --restart unless-stopped web:1.4.2   # mạng và chính sách khởi động lại
docker ps                                             # container đang chạy
docker ps -a                                          # mọi container, kể cả đã dừng
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"    # định dạng đầu ra
docker logs web                                       # log
docker logs -f --tail 100 web                         # theo dõi log, 100 dòng cuối
docker logs --since 10m web                           # log 10 phút gần nhất
docker exec -it web sh                                # shell trong container đang chạy
docker exec web cat /etc/nginx/nginx.conf             # chạy một lệnh
docker inspect web                                    # chi tiết cấu hình và trạng thái
docker inspect -f '{{.State.ExitCode}}' web           # chỉ lấy mã thoát
docker stats                                          # CPU, bộ nhớ, mạng, I/O theo thời gian thực
docker top web                                        # tiến trình trong container
docker cp web:/etc/nginx/nginx.conf ./nginx.conf      # sao chép từ container ra máy
docker cp ./config.txt web:/app/                      # sao chép từ máy vào container
docker stop web                                       # dừng nhẹ
docker kill web                                       # dừng ngay
docker restart web                                    # khởi động lại
docker pause web / docker unpause web                 # tạm dừng và tiếp tục
docker rm web                                         # xóa container đã dừng
docker rm -f web                                      # xóa cưỡng bức, kể cả đang chạy
docker container prune                                # xóa mọi container đã dừng
docker update --memory=1g web                         # đổi giới hạn tài nguyên của container đang chạy
docker wait web                                       # chờ container dừng, in mã thoát
```

## 35. Volume

```sh
docker volume create du-lieu            # tạo volume
docker volume ls                        # liệt kê
docker volume inspect du-lieu           # chi tiết và vị trí
docker volume rm du-lieu                # xóa
docker volume prune                     # xóa volume không dùng
```

## 36. Network

```sh
docker network create app-net           # tạo mạng bridge
docker network ls                       # liệt kê
docker network inspect app-net          # subnet, container kết nối
docker network connect app-net web      # gắn container vào mạng
docker network disconnect app-net web   # gỡ khỏi mạng
docker network rm app-net               # xóa mạng không còn container
docker network prune                    # xóa mạng không dùng
```

## 37. Build và buildx

```sh
docker build -t web:1.4.2 .                              # build từ Dockerfile trong thư mục hiện tại
docker build -f docker/Dockerfile.prod -t web:prod .     # chỉ định Dockerfile khác
docker build --no-cache -t web:1.4.2 .                   # bỏ cache
docker build --target build -t web:build .               # dừng ở giai đoạn build (multi-stage)
docker build --build-arg VERSION=1.4.2 -t web:1.4.2 .    # truyền ARG
docker buildx ls                                         # builder hiện có
docker buildx create --use --name multi                  # tạo builder mới và dùng
docker buildx build --platform linux/amd64,linux/arm64 -t reg/web:1.4.2 --push .   # đa nền tảng
docker buildx imagetools inspect reg/web:1.4.2           # xem manifest đa nền tảng [Trí nhớ]
docker builder prune                                     # xóa cache build (thận trọng)
```

## 38. Registry và đăng nhập

```sh
docker login registry.example.com        # đăng nhập
docker logout registry.example.com       # đăng xuất
docker push reg/web:1.4.2                # đẩy
docker pull reg/web:1.4.2                # kéo
docker manifest inspect reg/web:1.4.2    # xem manifest [Trí nhớ]
```

## 39. Compose

```sh
docker compose up -d                     # khởi động nền
docker compose up -d --build             # build lại rồi khởi động
docker compose down                      # dừng và xóa container, mạng
docker compose down -v                   # xóa cả volume
docker compose ps                        # trạng thái
docker compose logs -f                   # log toàn bộ dịch vụ
docker compose exec web sh               # shell trong dịch vụ
docker compose run --rm web pytest       # chạy lệnh một lần trong dịch vụ
docker compose config                    # xem cấu hình đã được tổng hợp, kiểm tra cú pháp
docker compose pull                      # kéo image mới nhất theo tag
docker compose restart web               # khởi động lại một dịch vụ
docker compose stop db                   # dừng một dịch vụ
```

## 40. Dọn dẹp và dung lượng

```sh
docker system df                         # xem dung lượng đang dùng
docker system prune                      # xóa container dừng, mạng không dùng, image treo, cache
docker system prune -a --volumes         # xóa mạnh hơn, kể cả image không dùng và volume (rất thận trọng)
```

**Cảnh báo:** `--volumes` xóa dữ liệu. Chạy `docker system df` và kiểm tra trước khi xóa. [Suy luận]

---

# PHẦN J. GỠ LỖI

## 41. Quy trình gỡ lỗi

```
1. Container đang ở trạng thái nào?      docker ps -a
2. Mã thoát là gì?                       docker inspect -f '{{.State.ExitCode}}' <tên>
3. Log nói gì?                           docker logs <tên>   (và --tail, --since)
4. Cấu hình thật là gì?                  docker inspect <tên>
5. Mạng, cổng, volume có đúng không?     docker port, docker network inspect, docker inspect
6. Tài nguyên có đủ không?               docker stats, docker system df
7. Daemon có khỏe không?                 docker info, docker version
```

## 42. Lỗi build

| Triệu chứng | Nguyên nhân thường gặp | Kiểm tra |
|---|---|---|
| `failed to solve: ... not found` | Image nền sai tên hoặc tag | Kiểm tra `FROM`, thử `docker pull` |
| `COPY failed: file not found` | File không nằm trong ngữ cảnh build, hoặc bị `.dockerignore` loại | Kiểm tra đường dẫn và `.dockerignore` |
| `apt-get` hoặc `pip` lỗi mạng | Mạng build bị chặn, proxy | Kiểm tra proxy, thử lại |
| Build chậm, không tận dụng cache | Thứ tự chỉ thị sai (copy mã trước khi cài phụ thuộc) | Đưa `COPY` phụ thuộc lên trước |
| Lệnh `RUN` thất bại | Lệnh sai trong image nền (ví dụ thiếu công cụ) | Chạy thử: `docker run --rm -it <image-nền> sh` |
| Image quá lớn | Công cụ build còn trong image chạy | Dùng multi-stage, `.dockerignore` |

```sh
docker build --progress=plain -t web:dev .    # xem chi tiết từng bước build
docker build --no-cache --progress=plain -t web:dev .   # build lại không cache để tái hiện lỗi
```

## 43. Lỗi container không khởi động hoặc thoát

```sh
docker ps -a                             # trạng thái Exited và mã thoát
docker logs <tên>                        # lỗi khởi động
docker inspect <tên> | grep -A 5 State   # trạng thái chi tiết
docker run -it --entrypoint sh web:1.4.2 # ghi đè entrypoint để vào shell thử nghiệm
```

| Mã thoát | Ý nghĩa thường gặp | Hướng xử lý |
|---|---|---|
| 0 | Lệnh chạy xong | Nếu là dịch vụ, lệnh có thể không giữ tiến trình chạy |
| 1 | Lỗi ứng dụng chung | Đọc log |
| 126 | Không có quyền thực thi | Kiểm tra quyền file và `USER` |
| 127 | Không tìm thấy lệnh | Sai `CMD`/`ENTRYPOINT` hoặc thiếu binary |
| 137 | Bị SIGKILL, thường do OOM | Kiểm tra `docker inspect` có `OOMKilled`, tăng `--memory` |
| 139 | Segfault | Lỗi trong ứng dụng hoặc thư viện |
| 143 | SIGTERM | Dừng bình thường; kiểm tra xử lý tín hiệu |

**Nguyên nhân khác:**
- Tiến trình chính là shell hoặc lệnh kết thúc, nên container dừng ngay.
- `CMD` dạng shell không chuyển tín hiệu đúng, nên `docker stop` chờ rồi mới kill.
- Biến môi trường hoặc file cấu hình thiếu.

```sh
docker inspect -f '{{.State.OOMKilled}}' <tên>       # kiểm tra OOM
```

## 44. Lỗi mạng và cổng

```sh
docker ps --format "table {{.Names}}\t{{.Ports}}"    # xem cổng đã publish
docker port <tên>                                    # ánh xạ cổng
docker network inspect <mạng>                        # container nào trong mạng
docker exec <tên> ping -c 2 db                       # kiểm tra phân giải và kết nối (nếu có ping)
docker run --rm --network <mạng> busybox nslookup db # kiểm tra DNS trong mạng
```

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| `connection refused` từ máy | Chưa publish cổng, hoặc ứng dụng nghe `127.0.0.1` trong container | Publish cổng; ứng dụng nghe `0.0.0.0` |
| Container không gọi được nhau bằng tên | Dùng mạng bridge mặc định, không phải mạng do người dùng tạo | Tạo mạng riêng và gắn cả hai |
| Cổng đã bị dùng | Xung đột cổng trên máy chủ | `docker ps`, hoặc đổi cổng publish |
| Mạng không truy cập ra ngoài | Quy tắc tường lửa, proxy, DNS | Kiểm tra `docker run --rm busybox ping` và DNS |

**Lưu ý:** ứng dụng nghe `127.0.0.1` bên trong container không nhận được lưu lượng từ bên ngoài container, kể cả khi đã publish cổng. Phải nghe `0.0.0.0`. [Suy luận từ mô hình mạng]

## 45. Lỗi volume và quyền

```sh
docker volume inspect <tên>             # vị trí và thông tin volume
docker run --rm -v <tên>:/data busybox ls -la /data    # xem quyền dữ liệu trong volume
docker exec <tên> id                    # UID và GID đang chạy
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `Permission denied` khi ghi | Người dùng trong container không có quyền | Đặt quyền volume hoặc `chown` trong Dockerfile, hoặc chạy đúng UID |
| Dữ liệu mất sau khi xóa container | Không dùng volume | Gắn volume có tên |
| Bind mount trống | Đường dẫn máy chủ sai | Kiểm tra đường dẫn tuyệt đối |
| Trên Windows/macOS chậm | Bind mount qua lớp ảo hóa | Dùng volume có tên cho dữ liệu nhiều I/O |

## 46. Lỗi registry và kéo image

| Lỗi | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `manifest unknown` | Tag không tồn tại | Kiểm tra tên và tag |
| `unauthorized` | Chưa đăng nhập hoặc không có quyền | `docker login`, kiểm tra quyền |
| `toomanyrequests` | Vượt giới hạn kéo của registry | Đăng nhập, dùng registry riêng, cache |
| `no matching manifest for linux/arm64` | Image không có bản cho kiến trúc | Build đa nền tảng, hoặc chọn image khác |
| Timeout khi kéo | Mạng, proxy, DNS | Kiểm tra mạng và cấu hình proxy |

```sh
docker manifest inspect <image>        # xem các nền tảng có sẵn [Trí nhớ]
docker info | grep -i -A 5 "proxy"     # kiểm tra cấu hình proxy
```

## 47. Lỗi tài nguyên và hiệu năng

```sh
docker stats --no-stream                # CPU, bộ nhớ hiện tại của mọi container
docker inspect -f '{{.HostConfig.Memory}}' <tên>    # giới hạn bộ nhớ (byte)
docker system df                        # dung lượng đĩa
```

**Dấu hiệu:**
- `OOMKilled=true`: bộ nhớ vượt giới hạn, tăng `--memory` hoặc tối ưu ứng dụng.
- CPU chạm giới hạn liên tục: tăng `--cpus` hoặc tối ưu.
- Đĩa đầy: `docker system df`, dọn dẹp có kiểm soát.

## 48. Lỗi daemon và Docker Desktop

```sh
docker info                             # nếu lỗi kết nối, daemon có thể chưa chạy
docker context ls                       # context đang trỏ đúng máy không
echo $DOCKER_HOST                       # biến môi trường có ghi đè không
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `Cannot connect to the Docker daemon` | Daemon chưa chạy, hoặc quyền socket | Khởi động Docker, kiểm tra quyền user trong nhóm `docker` (Linux) |
| Lệnh chạy được với sudo, không chạy được thường | Người dùng không trong nhóm `docker` | Thêm người dùng vào nhóm, đăng nhập lại (cần hiểu rủi ro quyền) |
| API version không tương thích | Client cũ so với daemon yêu cầu v1.44+ | Nâng cấp client |
| Docker Desktop không khởi động | Tài nguyên ảo hóa, cấu hình hệ điều hành | Xem nhật ký của Docker Desktop |

[Trang release notes xác nhận yêu cầu API v1.44+. Các bước khắc phục cụ thể là kiến thức nền]

---

# PHẦN K. QUAN SÁT VÀ VẬN HÀNH

## 49. Log, stats, events

- **Log:** container ghi ra stdout và stderr. Docker thu thập qua logging driver. Driver mặc định là `json-file`. [Trí nhớ]
- **Giới hạn log:** nên đặt giới hạn kích thước để tránh đầy đĩa.

```sh
docker run -d --log-opt max-size=10m --log-opt max-file=3 web:1.4.2   # giới hạn log xoay vòng
docker events --since 1h                 # sự kiện trong 1 giờ qua
docker stats                             # số liệu thời gian thực
```

## 50. Health check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \    # kiểm tra mỗi 30 giây, timeout 3 giây, thử 3 lần
  CMD curl -f http://localhost:8000/healthz || exit 1    # lệnh kiểm tra; lỗi trả mã khác 0
```

```sh
docker inspect --format '{{.State.Health.Status}}' web   # healthy / unhealthy / starting
```

**Lưu ý:** cần công cụ như `curl` có sẵn trong image. Image tối giản có thể không có. [Suy luận]

## 51. Cập nhật, sao lưu, và bảo trì

- **Cập nhật image:** build lại với image nền mới, kiểm thử, rồi triển khai. Ghim phiên bản, không dùng `latest`. [Suy luận]
- **Sao lưu volume:** chạy container tạm, nén dữ liệu, lưu ra ngoài.

```sh
docker run --rm -v du-lieu-db:/data -v "$(pwd):/backup" busybox \
  tar czf /backup/du-lieu-db.tar.gz -C /data .          # nén volume ra máy
docker run --rm -v du-lieu-db:/data -v "$(pwd):/backup" busybox \
  tar xzf /backup/du-lieu-db.tar.gz -C /data            # khôi phục volume
```

- **Sao lưu cơ sở dữ liệu:** nên dùng công cụ của chính cơ sở dữ liệu (ví dụ `pg_dump`) thay vì sao chép thư mục đang ghi. [Suy luận]

---

# PHẦN L. THỰC HÀNH SẢN XUẤT

Danh sách kiểm tra trước khi đưa container vào sản xuất:

- [ ] Image nền từ nguồn tin cậy, có tag cụ thể và digest đã ghim.
- [ ] Dockerfile multi-stage, không có công cụ build trong image chạy.
- [ ] Chạy bằng người dùng không phải root (`USER`).
- [ ] Hệ tệp chỉ đọc khi có thể (`--read-only`), volume cho dữ liệu cần ghi.
- [ ] Không có bí mật trong Dockerfile, `ENV`, `ARG`, hoặc image.
- [ ] `.dockerignore` loại bỏ `.git`, `.env`, và file không cần thiết.
- [ ] Giới hạn tài nguyên (`--memory`, `--cpus`) và giới hạn log.
- [ ] Quét lỗ hổng trong CI.
- [ ] Health check đúng và có cơ chế khởi động lại.
- [ ] Cổng publish tối thiểu, bind đúng IP.
- [ ] Sao lưu volume và kiểm tra khôi phục.
- [ ] Phiên bản Docker Engine và Compose được ghi lại và có kế hoạch nâng cấp.

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Image | Mẫu chỉ đọc để tạo container |
| Layer | Lớp của image, tạo bởi mỗi chỉ thị Dockerfile |
| Container | Instance đang chạy hoặc đã dừng của image |
| Dockerfile | Kịch bản mô tả cách build image |
| Tag / Digest | Nhãn phiên bản (có thể đổi) / mã băm nội dung (cố định) |
| Registry | Nơi lưu và phân phối image |
| Daemon (dockerd) | Tiến trình nền quản lý đối tượng Docker |
| containerd | Quản lý vòng đời container ở mức thấp |
| runc | Chạy container theo đặc tả OCI |
| BuildKit / buildx | Engine build và plugin CLI |
| Volume | Lưu trữ do Docker quản lý, bền vững |
| Bind mount | Gắn thư mục cụ thể của máy chủ |
| tmpfs | Hệ tệp trong RAM |
| Bridge / host / none / overlay | Các loại mạng |
| Publish (`-p`) | Ánh xạ cổng máy chủ đến container |
| Multi-stage build | Build nhiều giai đoạn, chỉ giữ kết quả cần thiết |
| `.dockerignore` | Loại file khỏi ngữ cảnh build |
| Compose | Công cụ chạy ứng dụng nhiều container |
| Health check | Lệnh kiểm tra sức khỏe container |
| Rootless | Daemon chạy không cần quyền root |
| OCI | Chuẩn định dạng image và runtime |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **Docker Overview (kiến trúc)**: https://docs.docker.com/get-started/docker-overview/ [đã đọc. Ghi nhận: client-server, REST API qua Unix socket hoặc mạng; dockerd, Docker client, Compose, Docker Desktop, registry, image, container, volume, network]
- **Tham chiếu CLI**: https://docs.docker.com/reference/cli/docker/ [đã đọc. Ghi nhận: các nhóm lệnh quản lý; `docker run`, `build`, `ps`, `exec`, `attach`, `start`. Không có phiên bản cho chính trang]
- **Release notes của Docker Engine**: https://docs.docker.com/engine/release-notes/ [đã đọc một phần. Ghi nhận: bản mới nhất được liệt kê là 29.9.2 (2026-10-20); không có nhãn stable; API tối thiểu v1.44; cgroup v1 đến ít nhất 2029; containerd và BuildKit đi kèm từng bản]

## N.2. Nguồn chưa đối chiếu

- Tài liệu Dockerfile reference (đầy đủ các chỉ thị, ENTRYPOINT và CMD).
- Tài liệu Compose (cú pháp `compose.yaml`, secrets, healthcheck, depends_on).
- Tài liệu mạng và lưu trữ chi tiết.
- Tài liệu Docker Scout, rootless mode, logging driver.
- Tài liệu BuildKit và buildx (cờ --secret, --platform).
- Tài liệu action của Docker cho CI.

## N.3. Điểm cần xác minh

1. **Phiên bản "stable" chính thức:** release notes không đánh dấu. Cần kiểm tra trang đã phát hành hoặc kênh của Docker trước khi chọn cho sản xuất.
2. **Phiên bản containerd và runc đi kèm:** lấy từ release notes; cần đối chiếu nếu dùng riêng.
3. **Cú pháp `docker debug`, `docker scout`, `docker init`, `docker mcp`:** có trong danh sách lệnh; chi tiết cần kiểm tra.
4. **Tên image và action trong ví dụ** (`gcr.io/distroless/static:nonroot`, `docker/build-push-action`) là minh họa; cần kiểm tra phiên bản hiện hành.
5. **Các mục đánh dấu `[Trí nhớ]`** (ENTRYPOINT/CMD, cache layer, mã thoát, logging driver mặc định, DNS trên mạng bridge do người dùng tạo) cần đối chiếu với tài liệu Dockerfile và Engine.
6. **Ví dụ shell và Dockerfile chưa được chạy thử.** Nên thử trên máy cục bộ trước khi dùng.
7. **Lệnh dọn dẹp** (`prune`, `system prune -a --volumes`) có thể xóa dữ liệu. Kiểm tra `docker system df` và sao lưu trước khi chạy.

---

**Tóm tắt để tra nhanh:**
- **Docker** = đóng gói ứng dụng thành image, chạy thành container cô lập chung nhân.
- **Kiến trúc:** CLI gọi REST API của dockerd; dockerd dùng containerd và runc; BuildKit build image; registry phân phối.
- **Đối tượng:** image (layer, tag, digest), container (vòng đời, lớp ghi riêng), volume, network.
- **Dockerfile:** thứ thay đổi ít đặt trước, multi-stage để giảm kích thước, không bí mật trong layer.
- **Mạng:** bridge mặc định; mạng do người dùng tạo có DNS theo tên; `-p` để publish, ứng dụng nghe `0.0.0.0`.
- **Lưu trữ:** volume cho dữ liệu bền vững, bind mount cho phát triển, tmpfs cho dữ liệu tạm.
- **Workflow:** build có tag, scan, push, Compose cho nhiều dịch vụ, CI tự động.
- **Gỡ lỗi:** `ps -a` → `ExitCode` → `logs` → `inspect` → `stats`; lỗi build theo chỉ thị; lỗi mạng theo cổng và tên mạng.
- **Cẩn trọng:** không mount socket vào container không tin cậy, không `--privileged` nếu không cần, và kiểm tra trước khi `prune`.
