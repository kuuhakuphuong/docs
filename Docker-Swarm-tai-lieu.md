# Docker Swarm: tài liệu tổng quan về mô hình, kiến trúc, đối tượng, mạng, workflow, câu lệnh, và gỡ lỗi

> **Viết cho:** người đã biết Docker và muốn chạy ứng dụng trên nhiều máy với công cụ có sẵn trong Docker Engine. Cấu trúc tương tự tài liệu Kubernetes để dễ đối chiếu.
> **Cách đọc:** Phần A đến C là mô hình và kiến trúc. Phần D đến F là mạng, lưu trữ, cấu hình, và lập lịch. Phần I là tham chiếu lệnh. Phần J là gỡ lỗi. Các đoạn lệnh và YAML có chú thích từng dòng bên cạnh.
> **Phiên bản tham chiếu:** Swarm mode có sẵn trong Docker Engine, không cài riêng. Phiên bản Engine hiện tại đã đối chiếu trong tài liệu Docker là 29.x (xem tài liệu Docker đã viết).
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với docs.docker.com; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các lệnh và file stack chưa được chạy thử trong lần này. Nên thử trên vài máy ảo trước khi dùng cho dự án thật.

---

## Mục lục

- **Phần A. Docker Swarm là gì và giải quyết vấn đề gì**
  - 1. Từ một máy đến nhiều máy
  - 2. Swarm mode trong Docker Engine
  - 3. Swarm và các lựa chọn khác
- **Phần B. Kiến trúc cluster**
  - 4. Manager và worker
  - 5. Raft: đồng thuận và quorum
  - 6. Scheduler, dispatcher, và reconciliation
  - 7. Chứng chỉ TLS và join token
  - 8. Luồng một lệnh `docker service create` đi qua các thành phần
- **Phần C. Các đối tượng cốt lõi**
  - 9. Node
  - 10. Service: replicated và global
  - 11. Task và vòng đời
  - 12. Stack: triển khai nhiều service bằng một file
  - 13. Overlay network
  - 14. Secret và config
- **Phần D. Mạng và kết nối**
  - 15. Overlay network và VXLAN
  - 16. Publish cổng và định tuyến
  - 17. DNS nội bộ và tên service
  - 18. Cổng cần mở
- **Phần E. Lưu trữ và dữ liệu**
  - 19. Volume trên swarm
  - 20. Dữ liệu có trạng thái (stateful)
- **Phần F. Lập lịch và tài nguyên**
  - 21. Resource: reservations và limits
  - 22. Placement: constraints và preferences
  - 23. Chế độ sẵn sàng của node (availability)
- **Phần G. Bảo mật**
  - 24. Mã hóa giữa các node
  - 25. Join token và xoay vòng
  - 26. Khóa tự động (autolock)
  - 27. Quản lý secret và config
- **Phần H. Workflow**
  - 28. Workflow triển khai
  - 29. Cập nhật theo đợt, rollback
  - 30. Quy trình vận hành hàng ngày
  - 31. Tích hợp CI/CD
- **Phần I. Câu lệnh**
  - 32. Quản lý swarm
  - 33. Quản lý node
  - 34. Quản lý service
  - 35. Quản lý stack
  - 36. Secret và config
  - 37. Mạng và volume
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 38. Quy trình gỡ lỗi
  - 39. Quorum và manager
  - 40. Node không tham gia hoặc mất kết nối
  - 41. Service ở trạng thái pending hoặc không chạy
  - 42. Task thất bại liên tục
  - 43. Mạng overlay và cổng
  - 44. Secret, config, và quyền
  - 45. Cập nhật và rollback không như mong đợi
- **Phần K. Vận hành**
  - 46. Bảo trì node và drain
  - 47. Sao lưu và khôi phục trạng thái swarm
  - 48. Nâng cấp
- **Phần L. So sánh với Kubernetes**
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. DOCKER SWARM LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Từ một máy đến nhiều máy

Khi ứng dụng chạy bằng Docker trên một máy, các câu hỏi đặt ra khi nhiều máy: container nào chạy ở máy nào, máy hỏng thì ai khởi động lại container, tải tăng thì ai thêm bản sao, và cập nhật phiên bản thế nào để không ngừng dịch vụ. [Suy luận]

**Docker Swarm** là chế độ điều phối (orchestration) có sẵn trong Docker Engine, cho phép gom nhiều máy thành một **cluster** và triển khai ứng dụng lên đó bằng các lệnh Docker quen thuộc. [Đã kiểm chứng: trang swarm mode mô tả Swarm mode được tích hợp trong Docker Engine và có thể tạo, quản lý bằng Docker CLI]

## 2. Swarm mode trong Docker Engine

- Swarm mode được **tích hợp** trong Docker Engine, không cần cài thêm phần mềm.
- Mặc định **tắt**. Bật bằng `docker swarm init` hoặc tham gia bằng `docker swarm join`. [Đã kiểm chứng]
- Trang Docker khuyến nghị dùng Swarm mode cho môi trường sản xuất nếu có kế hoạch dùng Swarm làm nền tảng chạy. Nếu chỉ chạy trên một máy, Docker Compose đủ. [Đã kiểm chứng]
- Tài liệu cảnh báo không nhầm với **Docker Classic Swarm** (đã ngừng). [Đã kiểm chứng]

## 3. Swarm và các lựa chọn khác

| Tiêu chí | Docker Compose | Docker Swarm | Kubernetes |
|---|---|---|---|
| Phạm vi | Một máy (thường) | Nhiều máy | Nhiều máy |
| Cài đặt | Có sẵn với Docker | Có sẵn với Docker Engine | Cần cài thêm và cấu hình phức tạp hơn |
| Độ phức tạp | Thấp | Trung bình | Cao |
| Hệ sinh thái | Docker | Docker | Rất lớn, nhiều công cụ |
| Phù hợp | Phát triển, ứng dụng nhỏ | Nhóm nhỏ, muốn đơn giản | Quy mô lớn, nhiều đội |

[Suy luận; so sánh khái quát. Trang Docker khuyến nghị Compose cho trường hợp một máy và Kubernetes cho nhu cầu tích hợp rộng]

---

# PHẦN B. KIẾN TRÚC CLUSTER

Cluster gồm **manager** (quyết định và lưu trạng thái) và **worker** (chạy container).

```
┌────────────── MANAGER (số lẻ, khuyến nghị 3 hoặc 5) ──────────────┐
│  Raft: đồng bộ trạng thái giữa các manager                        │
│  Scheduler (chọn node) · Dispatcher · Orchestrator (điều hòa)     │
│  API Docker (cổng 2377) · Lưu trạng thái cluster                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │  (mTLS)
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
   ┌─────── WORKER ───────┐ ┌─────── WORKER ───────┐ ┌─────── WORKER ───────┐
   │ dockerd              │ │ dockerd              │ │ dockerd              │
   │ task (container)     │ │ task (container)     │ │ task (container)     │
   │ task (container)     │ │ task (container)     │ │                      │
   └──────────────────────┘ └──────────────────────┘ └──────────────────────┘
```

Manager cũng có thể chạy task theo mặc định, nhưng trang Docker cảnh báo chúng nhạy với thiếu tài nguyên. Có thể drain manager để tách việc chạy task. [Đã kiểm chứng]

## 4. Manager và worker

| Vai trò | Trách nhiệm | Ghi chú |
|---|---|---|
| **Manager** | Quản lý trạng thái cluster, lập lịch, điều hòa, nhận lệnh API | Số lượng cần lẻ và đủ để giữ quorum |
| **Worker** | Chạy task (container) theo phân công | Có thể thêm bất kỳ lúc nào |

Trang Docker cho biết: bạn có thể triển khai cả manager và worker, và Engine tự xử lý vai trò khi chạy. Một image đĩa có thể dùng cho mọi node. [Đã kiểm chứng]

**Leader manager:** trong số các manager, một manager là leader để điều phối thay đổi. Nếu leader hỏng, các manager khác bầu leader mới. [Trí nhớ]

## 5. Raft: đồng thuận và quorum

Swarm dùng thuật toán đồng thuận **Raft** để các manager thống nhất trạng thái. Thay đổi cần được **đa số manager** đồng ý. [Đã kiểm chứng]

**Công thức quorum:** cần `floor(N/2) + 1` manager để đồng ý. Cluster chịu được tối đa `(N-1)/2` manager hỏng vĩnh viễn. [Đã kiểm chứng]

| Số manager (N) | Đa số cần | Chịu được hỏng |
|:---:|:---:|:---:|
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 6 | 4 | 2 |
| 7 | 4 | 3 |
| 8 | 5 | 3 |
| 9 | 5 | 4 |

[Đã kiểm chứng: bảng và công thức từ trang admin guide]

**Khuyến nghị:**
- Dùng **số lẻ** manager và **nhiều hơn hai**. Số chẵn không tăng khả năng chịu lỗi so với số lẻ thấp hơn một đơn vị. [Đã kiểm chứng]
- Phân bố manager trên **ít nhất ba vùng sẵn sàng (availability zone)**. Ví dụ: 3 manager là 1-1-1; 5 manager là 2-2-1. [Đã kiểm chứng]
- **Hai manager** không có lợi: nếu một hỏng, đa số không còn. [Suy luận từ công thức]

**Mất quorum:** cluster không thể thay đổi trạng thái (tạo, cập nhật service, thêm node). Các task đang chạy vẫn chạy, nhưng quản lý bị đóng băng. [Trí nhớ]

## 6. Scheduler, dispatcher, và reconciliation

- **Scheduler:** chọn node cho task mới, xét tài nguyên, ràng buộc, và trạng thái node. [Trí nhớ]
- **Orchestrator:** so sánh trạng thái mong muốn với trạng thái thực tế, tạo hoặc xóa task để khớp. [Đã kiểm chứng: trang service mô tả reconciliation]
- **Dispatcher:** gửi phân công task đến worker và nhận báo cáo trạng thái. [Trí nhớ]

**Vòng điều hòa:** nếu bạn yêu cầu 3 bản sao và một task chết, orchestrator tạo task mới trên node khác để đủ 3. Bạn không cần quản lý từng task. [Đã kiểm chứng]

## 7. Chứng chỉ TLS và join token

- Mọi node trong swarm dùng **mTLS** (xác thực hai chiều) và mã hóa giao tiếp. Có thể dùng chứng chỉ tự ký hoặc CA riêng. [Đã kiểm chứng]
- Khi `swarm init`, Docker tạo CA nội bộ và cấp chứng chỉ cho node. [Trí nhớ]
- **Join token** tách riêng cho manager và worker. Node dùng token để gia nhập. [Đã kiểm chứng]

## 8. Luồng một lệnh `docker service create` đi qua các thành phần

```
1. CLI gửi yêu cầu tạo service đến manager (API)
2. Manager lưu đặc tả service vào trạng thái Raft (đồng thuận)
3. Orchestrator tạo các task tương ứng số replica
4. Scheduler chọn node cho từng task (theo tài nguyên và ràng buộc)
5. Dispatcher gửi phân công đến worker
6. dockerd trên worker kéo image, tạo và chạy container
7. Worker báo trạng thái ngược về manager
```

[Suy luận từ mô hình đã kiểm chứng; các bước là minh họa]

---

# PHẦN C. CÁC ĐỐI TƯỢNG CỐT LÕI

## 9. Node

Node là máy tham gia swarm. Mỗi node có: vai trò (manager hoặc worker), availability (active, pause, drain), nhãn (label) và tài nguyên. [Trí nhớ]

```sh
docker node ls                              # danh sách node
docker node inspect <node> --pretty         # chi tiết dễ đọc
```

## 10. Service: replicated và global

**Service** là cách bạn triển khai ứng dụng: chỉ định image, lệnh, cổng, mạng, giới hạn tài nguyên, chính sách cập nhật, và số bản sao. [Đã kiểm chứng]

| Chế độ | Ý nghĩa | Dùng cho |
|---|---|---|
| **replicated** (mặc định) | Số task cố định, ví dụ 3 bản sao | Ứng dụng web, API |
| **global** | Mỗi node có đúng một task. Node mới tham gia sẽ tự có task | Agent giám sát, quét bảo mật |

[Đã kiểm chứng: trang service mô tả hai chế độ và ví dụ tương ứng]

```sh
docker service create --name web --replicas 3 -p 8080:80 nginx:1.27          # replicated, 3 bản sao
docker service create --name agent --mode global monitor-agent:1.0           # chạy trên mọi node
```

## 11. Task và vòng đời

**Task** là đơn vị lập lịch cơ bản. Mỗi task chạy **đúng một container**. Task là "ô" mà scheduler lấp đầy. [Đã kiểm chứng]

**Trạng thái task** (ví dụ): `NEW`, `PENDING`, `ASSIGNED`, `PREPARING`, `STARTING`, `RUNNING`, `COMPLETE`, `FAILED`, `SHUTDOWN`, `REJECTED`. [Đã kiểm chứng một phần: trang nêu ASSIGNED, PREPARING, RUNNING; các trạng thái khác là kiến thức nền]

Khi task thất bại, orchestrator xóa task và container, rồi tạo task thay thế. [Đã kiểm chứng]

**Service ở trạng thái `pending`:** xảy ra khi không node nào chạy được task, ví dụ tất cả node bị pause hoặc drain, hoặc không đáp ứng tài nguyên hay ràng buộc. Nếu chỉ muốn dừng triển khai, scale về 0. [Đã kiểm chứng]

## 12. Stack: triển khai nhiều service bằng một file

**Stack** là nhóm các service được mô tả trong một tệp YAML theo cú pháp Compose, mở rộng thêm khóa `deploy`. [Trí nhớ]

```yaml
version: "3.9"                             # phiên bản cú pháp (Compose spec có thể bỏ khóa này)
services:                                  # danh sách service
  web:                                     # tên service
    image: registry.example.com/web:1.4.2  # image có tag cụ thể
    ports:
      - "8080:80"                          # publish cổng: 8080 của cluster → 80 của container
    networks:
      - app-net                            # gắn vào mạng overlay
    deploy:                                # khóa riêng cho Swarm
      replicas: 3                          # 3 bản sao
      update_config:
        parallelism: 1                     # cập nhật 1 task mỗi lần
        delay: 10s                         # chờ 10 giây giữa các đợt
        failure_action: rollback           # lỗi thì quay về
      restart_policy:
        condition: on-failure              # khởi động lại khi lỗi
      resources:
        limits:
          memory: 512M                     # giới hạn bộ nhớ
        reservations:
          cpus: "0.25"                     # dành riêng 0,25 CPU để lập lịch

  db:
    image: postgres:16
    deploy:
      placement:
        constraints:
          - node.labels.disk == ssd        # chỉ đặt lên node có label disk=ssd
    volumes:
      - db-data:/var/lib/postgresql/data   # volume (lưu trên node cụ thể, xem mục 19)
    secrets:
      - db_password                        # tham chiếu secret

networks:
  app-net:
    driver: overlay                        # mạng overlay trải qua nhiều node
    attachable: true                       # cho phép container độc lập gắn vào

volumes:
  db-data:

secrets:
  db_password:
    external: true                         # secret đã tạo sẵn bằng docker secret create
```

```sh
docker stack deploy -c stack.yml shop      # triển khai stack tên shop
docker stack services shop                 # trạng thái service trong stack
docker stack ps shop                       # task của stack
docker stack rm shop                       # gỡ stack
```

**Lưu ý:** `docker stack deploy` bỏ qua một số khóa chỉ dùng với Compose (như `build`). Image phải có sẵn trong registry để các node kéo về. [Trí nhớ]

## 13. Overlay network

**Overlay network** cho phép container trên nhiều node nói chuyện như cùng một mạng. Swarm tự cấp địa chỉ và phân giải tên. [Đã kiểm chứng]

```sh
docker network create --driver overlay --attachable app-net   # tạo overlay có thể gắn từ bên ngoài service
docker network ls                                             # xem mạng (scope swarm)
```

Subnet mặc định của overlay lấy từ pool `10.0.0.0/8`. Có thể đổi bằng `--default-addr-pool` khi `swarm init`, để tránh xung đột với mạng nội bộ. [Đã kiểm chứng]

## 14. Secret và config

- **Secret:** dữ liệu nhạy cảm (mật khẩu, khóa). Được lưu mã hóa trong trạng thái swarm và chỉ gửi đến task cần dùng. Trong container, secret được mount dưới dạng tệp trong `/run/secrets/`. [Trí nhớ]
- **Config:** cấu hình không nhạy cảm (tệp cấu hình, mẫu). Cũng được mount vào container. [Trí nhớ]

```sh
printf 'mat-khau-manh' | docker secret create db_password -    # tạo secret từ stdin
docker config create nginx_conf ./nginx.conf                    # tạo config từ tệp
```

---

# PHẦN D. MẠNG VÀ KẾT NỐI

## 15. Overlay network và VXLAN

Overlay network đóng gói lưu lượng giữa các node qua đường hầm. Docker dùng **VXLAN** làm cơ chế đóng gói. [Trí nhớ]

**Cổng cần mở giữa các node (theo kiến thức nền, cần đối chiếu):** [Trí nhớ]
- **2377/TCP:** giao tiếp quản lý swarm (manager).
- **7946/TCP và UDP:** trao đổi thông tin node (gossip).
- **4789/UDP:** lưu lượng overlay (VXLAN).

**Lưu ý:** trang Docker đã đọc chỉ xác nhận cổng 2377. Hai cổng còn lại cần kiểm tra trong tài liệu mạng của swarm. [Đã kiểm chứng một phần]

## 16. Publish cổng và định tuyến

Khi publish cổng service, lưu lượng vào **bất kỳ node nào** trên cổng đó đều được chuyển đến một task của service, dù task không chạy trên node đó. Cơ chế này gọi là **routing mesh**. [Trí nhớ; trang Docker đã đọc không nêu tên routing mesh]

```sh
docker service create --name web --publish published=8080,target=80 --replicas 3 nginx:1.27
curl http://<IP-bất-kỳ-node>:8080       # đến được task dù IP đó không có task
```

**Chế độ publish khác:** `mode=host` gắn cổng trực tiếp vào node chạy task, không qua routing mesh. [Trí nhớ]

## 17. DNS nội bộ và tên service

Trong overlay network, mỗi service có tên DNS. Tên này phân giải về **địa chỉ ảo** của service (VIP), và swarm cân bằng tải giữa các task. [Đã kiểm chứng: trang service nói có DNS server nhúng để phân giải tên, và cân bằng tải qua cổng]

```sh
docker run --rm --network app-net busybox nslookup web    # tên service web trong mạng app-net
```

**Ghi chú:** chế độ `dnsrr` (DNS round robin) trả về IP của từng task thay vì VIP. Dùng khi cần tự cân bằng tải. [Trí nhớ]

## 18. Cổng cần mở trên tường lửa

Tóm tắt tương tự mục 15. Khi bật tường lửa giữa các node, mở đủ ba nhóm cổng (2377, 7946, 4789), và cổng publish của service. [Trí nhớ]

---

# PHẦN E. LƯU TRỮ VÀ DỮ LIỆU

## 19. Volume trên swarm

Volume có tên trong Docker là **cục bộ theo node**. Task có thể được lập lịch sang node khác, và khi đó sẽ không thấy dữ liệu của node cũ. [Suy luận từ mô hình volume cục bộ]

**Lựa chọn:**
- Ràng buộc service vào một node để dữ liệu ở đúng chỗ (đơn giản, nhưng mất tính sẵn sàng).
- Dùng volume driver phân tán hoặc lưu trữ mạng (NFS, dịch vụ cloud) cho dữ liệu cần di chuyển giữa node. [Trí nhớ]
- Dùng cơ sở dữ liệu có sao chép riêng (ví dụ replica của PostgreSQL) thay vì tự chia sẻ volume. [Suy luận]

## 20. Dữ liệu có trạng thái (stateful)

- **Không** chạy cơ sở dữ liệu có nhiều replica chỉ bằng `replicas` mà không có cơ chế sao chép riêng. Nhiều bản sao có thể ghi xung đột dữ liệu. [Suy luận]
- Với dữ liệu quan trọng, ràng buộc node và sao lưu thường xuyên. [Suy luận]

---

# PHẦN F. LẬP LỊCH VÀ TÀI NGUYÊN

## 21. Resource: reservations và limits

```sh
docker service create --name api \
  --reserve-cpu 0.5 --reserve-memory 256m \       # dành riêng để lập lịch
  --limit-cpu 1 --limit-memory 512m \             # giới hạn tối đa
  --replicas 2 api:1.0
```

- **Reservation:** tài nguyên dành trước khi lập lịch. Node không đủ sẽ không nhận task.
- **Limit:** trần tài nguyên khi chạy. [Trí nhớ]

## 22. Placement: constraints và preferences

```sh
docker node update --label-add zone=hanoi-a worker1          # gắn label cho node
docker service create --name web \
  --constraint 'node.labels.zone == hanoi-a' \                # chỉ đặt lên node này
  --constraint 'node.role != manager' \                       # không đặt lên manager
  --replicas 3 nginx:1.27
```

| Loại | Ví dụ | Ý nghĩa |
|---|---|---|
| Constraint | `node.role == worker` | Bắt buộc, task không đặt nếu không thỏa |
| Preference | `spread=node.labels.zone` | Ưu tiên phân tán đều giữa các giá trị |

[Trí nhớ; cú pháp constraint được Swarm hỗ trợ, cần kiểm tra phiên bản]

## 23. Chế độ sẵn sàng của node (availability)

| Trạng thái | Nhận task mới | Task đang chạy | Ghi chú |
|---|---|---|---|
| `active` | Có | Giữ nguyên | Bình thường |
| `pause` | Không | Giữ nguyên | Tạm dừng nhận việc mới |
| `drain` | Không | Chuyển đi nơi khác | Dùng để bảo trì |

```sh
docker node update --availability drain worker2     # sơ tán task khỏi node
docker node update --availability active worker2    # đưa lại vào hoạt động
```

---

# PHẦN G. BẢO MẬT

## 24. Mã hóa giữa các node

Mọi node dùng **mTLS**. Giao tiếp quản lý, dữ liệu secret, và lưu lượng overlay (theo cấu hình) được bảo vệ. [Đã kiểm chứng: trang swarm mode nói mỗi node thực thi mTLS và mã hóa]

Có thể dùng CA tự ký hoặc CA riêng. Chứng chỉ được xoay vòng định kỳ theo mặc định. [Trí nhớ]

## 25. Join token và xoay vòng

- **Join token** là chìa khóa để node gia nhập. Có hai loại, manager và worker. [Đã kiểm chứng]
- Xoay token khi: token bị lộ, một node bị xâm phạm, hoặc định kỳ ít nhất **sáu tháng một lần**. [Đã kiểm chứng]

```sh
docker swarm join-token worker              # xem lệnh và token gia nhập worker
docker swarm join-token manager             # xem lệnh và token gia nhập manager
docker swarm join-token --rotate worker     # xoay token worker, token cũ bị vô hiệu
```

**Lưu ý:** token trong lệnh `join` là thông tin nhạy cảm. Không đưa vào mã nguồn hoặc log công khai. [Suy luận]

## 26. Khóa tự động (autolock)

Autolock mã hóa khóa dữ liệu của manager trên đĩa. Khi manager khởi động lại, cần nhập **khóa mở** (unlock key) để tham gia lại. [Trí nhớ]

```sh
docker swarm update --autolock=true         # bật autolock, Docker in khóa mở
docker swarm unlock                         # nhập khóa mở sau khi khởi động lại
docker swarm unlock-key                     # xem hoặc xoay khóa mở
```

**Cảnh báo:** nếu mất khóa mở, manager có autolock không thể khởi động lại. Lưu khóa ở nơi an toàn, tách khỏi cluster. [Trí nhớ]

## 27. Quản lý secret và config

- Secret chỉ đưa đến task được cấp quyền, không ghi vào image.
- Không đặt mật khẩu trong `environment` của stack file. Dùng secret. [Suy luận]
- Config không mã hóa theo cùng cách secret. Đừng đặt dữ liệu nhạy cảm vào config. [Trí nhớ]

```sh
docker secret ls                            # liệt kê secret (không hiện nội dung)
docker secret inspect db_password           # xem metadata
docker service create --name app --secret db_password app:1.0   # gắn secret vào service
```

Trong container, đọc từ `/run/secrets/db_password`. [Trí nhớ]

---

# PHẦN H. WORKFLOW

## 28. Workflow triển khai

```
1. Chuẩn bị 3 hoặc 5 máy (manager và worker), mở cổng cần thiết
2. Khởi tạo swarm trên manager đầu tiên
3. Thêm manager còn lại bằng token manager (phân bố vùng sẵn sàng)
4. Thêm worker bằng token worker
5. Gắn label cho node theo vai trò, vùng, phần cứng
6. Tạo secret, config, và mạng overlay
7. Viết stack file, triển khai bằng docker stack deploy
8. Kiểm tra trạng thái service và task
9. Theo dõi, cập nhật theo đợt, rollback khi lỗi
```

```sh
# Trên manager đầu tiên
docker swarm init --advertise-addr 192.168.1.10          # khởi tạo; chỉ định IP quảng bá
docker swarm join-token manager                           # lấy lệnh thêm manager

# Trên máy khác (dán lệnh join nhận được)
docker swarm join --token <token> 192.168.1.10:2377      # tham gia cluster

# Kiểm tra
docker node ls                                            # danh sách node và vai trò
```

**Lưu ý:** `--advertise-addr` cần khi máy có nhiều IP. Địa chỉ này phải đến được từ mọi node khác. [Đã kiểm chứng]

## 29. Cập nhật theo đợt, rollback

Service có chính sách cập nhật: số task cập nhật cùng lúc, thời gian chờ giữa các đợt, và hành động khi lỗi. [Đã kiểm chứng: trang service nhắc đến rolling update policy; chi tiết là kiến thức nền]

```sh
docker service update --image nginx:1.28 web          # đổi image, kích hoạt rolling update
docker service update --update-parallelism 1 --update-delay 10s web   # cập nhật 1 task, chờ 10 giây
docker service update --update-failure-action rollback web            # lỗi thì tự quay lại
docker service rollback web                           # quay về cấu hình trước
docker service ps web                                 # xem lịch sử task
```

[Tên cờ cần kiểm tra theo `docker service update --help` của phiên bản đang dùng]

## 30. Quy trình vận hành hàng ngày

- Kiểm tra `docker node ls` và `docker service ls` định kỳ.
- Theo dõi task `FAILED` và `REJECTED`.
- Xem tài nguyên node, đĩa, và log.
- Cập nhật image có tag cụ thể, không dùng `latest`.
- Sao lưu trạng thái swarm (mục 47).

## 31. Tích hợp CI/CD

```yaml
name: deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          # Kết nối tới manager qua DOCKER_HOST (qua SSH hoặc TLS, không mở cổng 2377 ra Internet)
          docker stack deploy -c stack.yml shop      # triển khai hoặc cập nhật stack
        env:
          DOCKER_HOST: ${{ secrets.DOCKER_HOST }}    # lấy từ secret của CI
```

[Minh họa; cách kết nối an toàn cần đánh giá theo hạ tầng]

**Nguyên tắc:** không mở cổng API của manager ra Internet. Dùng SSH tunnel, VPN, hoặc TLS client certificate. [Suy luận]

---

# PHẦN I. CÂU LỆNH

## 32. Quản lý swarm

```sh
docker swarm init                          # khởi tạo swarm trên máy này (trở thành manager đầu tiên)
docker swarm init --advertise-addr <ip>    # chỉ định IP quảng bá
docker swarm join --token <tok> <ip>:2377  # tham gia cluster
docker swarm join-token worker             # xem lệnh và token gia nhập worker
docker swarm join-token manager            # xem lệnh và token gia nhập manager
docker swarm join-token --rotate worker    # xoay token worker
docker swarm leave                         # rời cluster (worker)
docker swarm leave --force                 # rời cluster (manager, thận trọng: có thể mất quorum)
docker swarm update --autolock=true        # bật autolock
docker swarm unlock                        # mở khóa manager sau khi khởi động lại
docker swarm unlock-key --rotate           # xoay khóa mở
docker info | grep -i swarm                # trạng thái swarm trên máy này
```

## 33. Quản lý node

```sh
docker node ls                                        # danh sách node
docker node ps                                        # task trên node hiện tại
docker node ps <node>                                 # task trên node cụ thể
docker node inspect <node> --pretty                   # chi tiết
docker node update --availability drain <node>        # sơ tán task
docker node update --availability active <node>       # đưa lại vào hoạt động
docker node update --availability pause <node>        # tạm không nhận task mới
docker node update --label-add zone=a <node>          # thêm label
docker node update --label-rm zone <node>             # xóa label
docker node promote <node>                            # nâng worker thành manager
docker node demote <node>                             # hạ manager thành worker
docker node rm <node>                                 # xóa node đã ngừng (sau khi leave)
```

## 34. Quản lý service

```sh
docker service create --name web --replicas 3 -p 8080:80 nginx:1.27   # tạo service
docker service create --name agent --mode global monitor:1.0          # chạy trên mọi node
docker service ls                                     # danh sách service
docker service ps web                                 # task của service
docker service ps web --no-trunc                      # task đầy đủ thông tin lỗi
docker service inspect web --pretty                   # chi tiết dễ đọc
docker service logs -f web                            # log hợp nhất của mọi task
docker service scale web=5                            # đổi số bản sao
docker service update --image nginx:1.28 web          # cập nhật image
docker service update --env-add MUC_LOG=info web      # thêm biến môi trường
docker service update --publish-add 9090:80 web       # thêm cổng
docker service rollback web                           # quay lại cấu hình trước
docker service rm web                                 # xóa service
```

## 35. Quản lý stack

```sh
docker stack deploy -c stack.yml shop      # triển khai hoặc cập nhật
docker stack ls                             # danh sách stack
docker stack services shop                  # service trong stack
docker stack ps shop                        # task trong stack
docker stack ps shop --filter desired-state=running   # chỉ task đang chạy
docker stack rm shop                        # gỡ stack
```

## 36. Secret và config

```sh
docker secret create <tên> <tệp-hoặc-->     # tạo secret (dấu - là đọc từ stdin)
docker secret ls                             # danh sách secret
docker secret inspect <tên>                  # metadata (không hiện nội dung)
docker secret rm <tên>                       # xóa secret không còn được dùng
docker config create <tên> <tệp>             # tạo config
docker config ls                             # danh sách config
docker config rm <tên>                       # xóa config
```

## 37. Mạng và volume

```sh
docker network create --driver overlay --attachable app-net   # overlay có thể gắn từ ngoài
docker network ls                                              # danh sách mạng
docker network inspect app-net                                 # chi tiết, subnet, container
docker network rm app-net                                      # xóa mạng không còn dùng
docker volume create <tên>                                     # tạo volume (cục bộ trên node)
docker volume ls                                               # danh sách volume
docker volume inspect <tên>                                    # chi tiết
```

---

# PHẦN J. GỠ LỖI

## 38. Quy trình gỡ lỗi

```
1. Cluster có khỏe không?         docker node ls, docker info
2. Quorum có còn không?           docker node ls (cột MANAGER STATUS), số manager còn Reachable
3. Service có chạy không?         docker service ls, docker service ps <tên>
4. Task lỗi gì?                   docker service ps <tên> --no-trunc (cột ERROR)
5. Log nói gì?                    docker service logs <tên>, docker logs <container> trên node
6. Mạng và cổng ổn không?         docker network inspect, kiểm tra cổng giữa các node
7. Tài nguyên có đủ không?        docker node inspect, docker stats trên node
```

## 39. Quorum và manager

```sh
docker node ls                                  # cột MANAGER STATUS: Leader, Reachable, Unavailable
docker info | grep -A 3 "Swarm"                 # trạng thái và số manager
```

**Dấu hiệu mất quorum:**
- Lệnh thay đổi cluster bị từ chối hoặc treo.
- `docker node ls` không chạy được trên manager còn lại (thông báo không có leader). [Trí nhớ]

**Xử lý:**
- Khôi phục manager hỏng nếu có thể (khởi động lại, sửa mạng).
- Nếu không khôi phục được, cần thao tác thủ công theo tài liệu mất quorum của Docker. Thao tác này có rủi ro mất dữ liệu trạng thái, nên sao lưu trước. [Trí nhớ]
- Không tự ý `docker swarm init --force-new-cluster` trên manager còn sống khi chưa hiểu hậu quả. [Trí nhớ]

## 40. Node không tham gia hoặc mất kết nối

```sh
docker node ls                                   # STATUS: Ready hoặc Down, AVAILABILITY
docker node inspect <node> --pretty              # trạng thái chi tiết
```

Trên node có lỗi (SSH):

```sh
systemctl status docker                          # daemon có chạy không
journalctl -u docker --since "30 min ago" | tail -n 100   # log daemon
ss -tlnp | grep -E '2377|7946'                   # cổng quản lý có lắng nghe không
```

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| `join` treo hoặc báo không kết nối | Cổng 2377 bị chặn, IP advertise sai | Kiểm tra tường lửa và `--advertise-addr` |
| Node `Down` | Daemon dừng, mạng đứt, hết đĩa | Khởi động daemon, kiểm tra mạng và đĩa |
| Lỗi chứng chỉ | Node cũ, đồng hồ lệch | Đồng bộ thời gian (NTP), tham gia lại bằng token mới |
| `invalid token` | Token đã xoay hoặc sai loại | Lấy lại token bằng `join-token` |

## 41. Service ở trạng thái pending hoặc không chạy

```sh
docker service ls                          # REPLICAS: 0/3 là dấu hiệu
docker service ps <tên> --no-trunc        # cột ERROR cho biết lý do
docker node ls                            # có node nào Active không
```

| Thông báo (cột ERROR) | Ý nghĩa | Hướng xử lý |
|---|---|---|
| `no suitable node (...)` | Không node nào thỏa constraint hoặc tài nguyên | Kiểm tra label, constraint, reservation |
| `insufficient resources` | Thiếu CPU hoặc bộ nhớ | Giảm reservation, thêm node |
| `image not found` / `pull access denied` | Không kéo được image | Kiểm tra tên, tag, đăng nhập registry |
| `no such secret` / `config not found` | Chưa tạo secret hoặc config | Tạo đối tượng còn thiếu |
| Mọi node `Drain` hoặc `Pause` | Không node nào nhận việc | Đưa ít nhất một node về `active` |

## 42. Task thất bại liên tục

```sh
docker service ps <tên> --no-trunc                    # lịch sử task và lỗi
docker service logs <tên> --tail 100                  # log các task
docker inspect <container-id>                         # trên node chạy task, xem ExitCode
```

**Mã thoát và nguyên nhân** tương tự Docker đơn máy (xem tài liệu Docker, mục 43). [Suy luận]

**Vòng lặp khởi động lại:** `restart_policy` quá tích cực có thể tạo task liên tục. Đặt giới hạn số lần hoặc sửa nguyên nhân. [Suy luận]

## 43. Mạng overlay và cổng

```sh
docker network inspect <mạng>              # container và subnet
docker run --rm --network <mạng> busybox nslookup <service>   # DNS trong mạng
docker run --rm --network <mạng> busybox wget -qO- http://<service>:<cổng>   # kết nối HTTP
```

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| Container trên node khác không liên lạc | Cổng 4789/UDP hoặc 7946 bị chặn | Mở cổng giữa các node |
| Tên service không phân giải | Container không cùng mạng overlay | Gắn đúng mạng |
| Publish cổng không đến được | Ứng dụng nghe `127.0.0.1`, hoặc tường lửa chặn | Ứng dụng nghe `0.0.0.0`, mở cổng |
| Trùng subnet với mạng nội bộ | Pool `10.0.0.0/8` xung đột | Đổi `--default-addr-pool` khi init (cần kế hoạch) |

## 44. Secret, config, và quyền

- Service không chạy vì thiếu secret: tạo secret trước khi deploy stack.
- `external: true` trong stack nghĩa là secret phải có sẵn trong cluster. [Trí nhớ]
- Ứng dụng không đọc được `/run/secrets/...`: kiểm tra tên file và quyền đọc của người dùng trong container. [Suy luận]

## 45. Cập nhật và rollback không như mong đợi

```sh
docker service ps <tên>                                  # task cũ và mới, trạng thái
docker service inspect <tên> --pretty | grep -A 10 Update   # cấu hình cập nhật
docker service rollback <tên>                            # quay lại
```

**Nguyên nhân thường gặp:**
- Image mới lỗi khởi động, task mới `FAILED` liên tục, rollout dừng.
- `update_config` đặt `parallelism` quá cao, làm mất hết bản sao cũ cùng lúc.
- Health check mới thất bại, task bị kill vòng lặp. [Suy luận]

---

# PHẦN K. VẬN HÀNH

## 46. Bảo trì node và drain

**Quy trình bảo trì worker:**
```sh
docker node update --availability drain <node>    # sơ tán task
docker service ps <service>                       # kiểm tra task đã sang node khác
# Thực hiện bảo trì trên node
docker node update --availability active <node>   # đưa lại vào hoạt động
```

**Bảo trì manager:** drain manager để tách khỏi việc chạy task. Khi tắt một manager, nhớ rằng quorum phải còn đủ. Ví dụ với 5 manager, có thể mất thêm một manager trong lúc bảo trì. [Đã kiểm chứng]

## 47. Sao lưu và khôi phục trạng thái swarm

Trạng thái swarm (Raft log, cấu hình, secret) nằm trên đĩa của **manager**, thường trong thư mục `/var/lib/docker/swarm`. [Trí nhớ]

**Nguyên tắc:**
- Sao lưu thư mục này từ một manager khi đã dừng, hoặc theo hướng dẫn của Docker để tránh tệp đang ghi. [Trí nhớ]
- Sao lưu stack file, secret (ngoài cluster), và định nghĩa mạng.
- Kiểm tra khôi phục trên cluster thử nghiệm.

```sh
sudo tar czf swarm-backup-$(date +%F).tar.gz /var/lib/docker/swarm    # sao lưu (cần dừng tiến trình thích hợp)
```

[Minh họa; cần đối chiếu hướng dẫn sao lưu chính thức trước khi dùng]

## 48. Nâng cấp

- Nâng cấp Docker Engine **từng node một**, giữ quorum manager trong suốt quá trình.
- Drain node trước khi nâng cấp, active lại sau khi xong.
- Nâng manager sau cùng hoặc theo thứ tự tài liệu của phiên bản. [Suy luận]
- Đọc ghi chú phát hành của Docker Engine trước khi nâng cấp. [Đã kiểm chứng: trang release notes đã đọc trong tài liệu Docker]

---

# PHẦN L. SO SÁNH VỚI KUBERNETES

| Khía cạnh | Docker Swarm | Kubernetes |
|---|---|---|
| Cài đặt | Có sẵn trong Docker Engine | Cần cài thêm và cấu hình nhiều thành phần |
| Đơn vị lập lịch | Task (một container) | Pod (một hoặc nhiều container) |
| Khai báo | Stack file (YAML theo Compose) | Manifest (nhiều loại tài nguyên) |
| Điều hòa | Orchestrator, reconciliation | Controller, reconciliation |
| Đồng thuận | Raft giữa manager | etcd (dữ liệu), kiến trúc khác |
| Mạng | Overlay, routing mesh | CNI, Service, Ingress/Gateway |
| Lưu trữ | Volume cục bộ, plugin | PV, PVC, StorageClass, CSI |
| Co giãn | Thủ công (`scale`), không có HPA tích hợp | HPA, VPA, PDB |
| Bảo mật | mTLS, secret, autolock | RBAC, NetworkPolicy, Pod Security |
| Hệ sinh thái | Nhỏ hơn | Rất lớn |
| Độ phức tạp | Thấp | Cao |
| Phù hợp | Nhóm nhỏ, cần đơn giản | Quy mô lớn, cần nhiều tính năng |

[Suy luận và kiến thức nền; so sánh khái quát. Các chi tiết của Swarm ở cột này có một số mục chưa đối chiếu đầy đủ]

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Swarm mode | Chế độ điều phối tích hợp trong Docker Engine |
| Node | Máy tham gia cluster |
| Manager | Node quản lý trạng thái và lập lịch |
| Worker | Node chạy task |
| Leader | Manager điều phối thay đổi tại một thời điểm |
| Raft | Thuật toán đồng thuận giữa manager |
| Quorum | Đa số manager cần để đồng ý thay đổi |
| Service | Đặc tả ứng dụng: image, cổng, số bản sao |
| Task | Đơn vị lập lịch, chạy đúng một container |
| Replicated / Global | Hai chế độ service |
| Stack | Nhóm service khai báo trong một file |
| Reconciliation | Điều hòa trạng thái thực tế về trạng thái mong muốn |
| Overlay network | Mạng trải qua nhiều node |
| VXLAN | Cơ chế đóng gói của overlay |
| Routing mesh | Định tuyến cổng publish đến task bất kỳ |
| VIP | Địa chỉ ảo của service |
| Secret / Config | Dữ liệu nhạy cảm / cấu hình không nhạy cảm |
| Join token | Chìa khóa để node gia nhập |
| mTLS | Xác thực và mã hóa hai chiều |
| Autolock | Mã hóa khóa dữ liệu manager, cần mở khóa khi khởi động |
| Availability | Trạng thái nhận việc của node: active, pause, drain |
| Constraint / Preference | Ràng buộc bắt buộc / ưu tiên đặt task |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **Docker Swarm mode (tổng quan)**: https://docs.docker.com/engine/swarm/ [đã đọc. Ghi nhận: tích hợp trong Engine; khuyến nghị cho sản xuất; node manager và worker; reconciliation; overlay; DNS nhúng; mTLS; rolling update; lệnh init và join]
- **Swarm mode, khái niệm cốt lõi và cấu hình**: https://docs.docker.com/engine/swarm/swarm-mode/ [đã đọc một phần. Ghi nhận: mặc định tắt; subnet overlay mặc định 10.0.0.0/8 và `--default-addr-pool`; join token riêng, xoay ít nhất sáu tháng một lần; cổng 2377]
- **Services (dịch vụ và tác vụ)**: https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/ [đã đọc. Ghi nhận: service, task, replicated, global, trạng thái pending]
- **Admin guide (quorum và vùng sẵn sàng)**: https://docs.docker.com/engine/swarm/admin_guide/ [đã đọc. Ghi nhận: công thức quorum, bảng chịu lỗi, số lẻ, phân bố ba vùng, drain manager]

## N.2. Nguồn chưa đối chiếu

- Tài liệu chi tiết về mạng swarm (cổng 7946 và 4789, routing mesh, chế độ publish).
- Tài liệu lệnh `docker service create` và `docker service update` (đầy đủ cờ).
- Tài liệu secret và config chi tiết, autolock, sao lưu và khôi phục trạng thái.
- Tài liệu constraint và preference đầy đủ.
- Tài liệu nâng cấp và xử lý mất quorum.

## N.3. Điểm cần xác minh

1. **Cổng 7946 (TCP/UDP) và 4789 (UDP):** trang admin guide đã đọc không liệt kê. Kiến thức nền đưa ra ba cổng; cần đối chiếu tài liệu mạng.
2. **Tên routing mesh và chế độ publish `host`:** kiến thức nền.
3. **Quy trình sao lưu thư mục swarm:** lệnh `tar` trong tài liệu là minh họa. Cần làm theo hướng dẫn chính thức để tránh sao lưu tệp đang ghi.
4. **Quy trình khôi phục khi mất quorum và `--force-new-cluster`:** rủi ro cao, cần đối chiếu tài liệu chính thức trước khi thực hiện.
5. **Cú pháp `docker service update` và tên cờ** (ví dụ `--update-failure-action`, `--update-parallelism`): cần kiểm tra theo phiên bản Engine.
6. **Vòng đời đầy đủ của task** (các trạng thái ngoài ASSIGNED, PREPARING, RUNNING): kiến thức nền.
7. **Ví dụ stack file và GitHub Actions** chưa được chạy thử. Khóa `version` trong Compose spec có thể không còn bắt buộc.
8. **So sánh với Kubernetes** là khái quát; một số ô dựa trên kiến thức nền.
9. **Trạng thái hỗ trợ lâu dài của Swarm:** chưa đối chiếu lộ trình phát triển hiện tại. Cần kiểm tra trước khi chọn cho dự án dài hạn.

---

**Tóm tắt để tra nhanh:**
- **Swarm** = điều phối tích hợp trong Docker Engine: manager (đồng thuận Raft) và worker (chạy task).
- **Quorum** = `floor(N/2)+1`; dùng số manager lẻ, tối thiểu 3, phân bố ít nhất ba vùng.
- **Đối tượng:** service (replicated hoặc global) → task (một container); stack để khai báo nhiều service.
- **Mạng:** overlay cho giao tiếp nhiều node, DNS theo tên service, publish cổng đến được từ mọi node.
- **Bảo mật:** mTLS mặc định, join token cần xoay định kỳ, autolock để bảo vệ khóa manager, secret thay cho biến môi trường.
- **Workflow:** `swarm init` → `join` → label → secret và mạng → `stack deploy` → `update` theo đợt → rollback khi lỗi.
- **Gỡ lỗi:** `node ls` (quorum và trạng thái node) → `service ps --no-trunc` (lỗi task) → `service logs` → kiểm tra mạng và cổng.
- **Cẩn trọng:** không mở cổng 2377 ra Internet, không tự ý `--force-new-cluster`, sao lưu trước khi thao tác với trạng thái manager.
