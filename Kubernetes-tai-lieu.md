# Kubernetes (K8s): tài liệu tổng quan về mô hình, kết nối, workflow, và câu lệnh

> **Viết cho:** người mới học Kubernetes đến người vận hành cluster. Tài liệu đi từ mô hình khái niệm, qua kiến trúc và các đối tượng, đến workflow thực tế, câu lệnh `kubectl`, và gỡ lỗi.
> **Cách đọc:** Phần A đến C cho mô hình nền. Phần D và E cho mạng và lưu trữ. Phần I là tham chiếu lệnh. Mỗi đoạn YAML và lệnh shell có chú thích từng dòng ngay bên cạnh.
> **Phiên bản tham chiếu:** trang releases chính thức (tra cứu 2026-10-21) cho thấy bản minor mới nhất là **1.37** (bản vá 1.37.1, 2026-09-15). Các bản 1.36 và 1.35 vẫn được hỗ trợ. 1.34 được hỗ trợ đến 2026-10-27.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với kubernetes.io; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các manifest và lệnh chưa được chạy thử. Trước khi dùng trên cluster thật, hãy thử trên cluster cục bộ (kind hoặc minikube) và kiểm tra `apiVersion` theo phiên bản của bạn.

---

## Mục lục

- **Phần A. Kubernetes là gì và giải quyết vấn đề gì**
  - 1. Container và vấn đề vận hành ở quy mô lớn
  - 2. Kubernetes làm gì
  - 3. Phiên bản và chính sách hỗ trợ
- **Phần B. Kiến trúc cluster**
  - 4. Control plane
  - 5. Node components
  - 6. Add-on và thành phần ngoài
  - 7. Luồng một yêu cầu đi qua các thành phần
- **Phần C. Mô hình đối tượng**
  - 8. Mô hình khai báo và vòng lặp điều hòa (reconciliation)
  - 9. Pod
  - 10. Label, selector, annotation
  - 11. ReplicaSet và Deployment
  - 12. StatefulSet, DaemonSet, Job, CronJob
  - 13. Namespace
- **Phần D. Mạng và kết nối**
  - 14. Mô hình mạng của Kubernetes
  - 15. Service và các loại Service
  - 16. DNS nội bộ
  - 17. Ingress và Gateway API
  - 18. NetworkPolicy
- **Phần E. Cấu hình và lưu trữ**
  - 19. ConfigMap và Secret
  - 20. Volume, PersistentVolume, PersistentVolumeClaim, StorageClass
- **Phần F. Lập lịch, tài nguyên, và tự co giãn**
  - 21. Requests và limits
  - 22. Probes: liveness, readiness, startup
  - 23. Lập lịch: nodeSelector, affinity, taint, toleration
  - 24. Tự co giãn: HPA, VPA, PDB
- **Phần G. Bảo mật**
  - 25. Xác thực, phân quyền, RBAC
  - 26. ServiceAccount
  - 27. Pod Security và bảo mật container
- **Phần H. Workflow**
  - 28. Workflow phát triển và triển khai
  - 29. Rolling update, rollback, và chiến lược triển khai
  - 30. Helm và Kustomize
  - 31. GitOps và CI/CD
- **Phần I. Câu lệnh kubectl**
  - 32. Cấu hình và ngữ cảnh
  - 33. Xem tài nguyên
  - 34. Tạo, sửa, xóa
  - 35. Chạy và gỡ lỗi
  - 36. Quản lý node
  - 37. Rollout và scale
  - 38. Mạng và truy cập
  - 39. Lệnh Helm thường dùng
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 40. Quy trình gỡ lỗi
  - 41. Các trạng thái Pod thường gặp
  - 42. Gỡ lỗi theo lớp: 42.1 năm câu hỏi đầu, 42.2 bản đồ triệu chứng, 42.3 Pending, 42.4 image và container, 42.5 readiness, 42.6 Terminating, 42.7 cấu hình im lặng, 42.8 Service, 42.9 PVC, 42.10 Ingress, 42.11 DNS, 42.12 node NotReady, 42.13 RBAC, 42.14 control plane, 42.15 tài nguyên, 42.16 bộ lệnh nhanh, 42.17 công cụ mạng
  - 43. Runbook sự cố: mẫu ghi chép
  - 44. Phòng ngừa
- **Phần K. Quan sát và vận hành**
  - 42. Logging, metrics, tracing
  - 43. Sao lưu, nâng cấp, và bảo trì node
- **Phần L. Môi trường học và thử nghiệm**
  - 44. Cluster cục bộ
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. KUBERNETES LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Container và vấn đề vận hành ở quy mô lớn

**Container** đóng gói ứng dụng cùng phụ thuộc, chạy nhất quán giữa các môi trường. Khi số container tăng lên hàng trăm hoặc hàng nghìn, cần trả lời các câu hỏi:
- Container nào chạy ở máy nào?
- Container chết thì ai khởi động lại?
- Tải tăng thì ai thêm bản sao?
- Hai dịch vụ tìm thấy nhau như thế nào?
- Cập nhật phiên bản mà không ngừng dịch vụ thì làm sao?

**Kubernetes (K8s)** là hệ thống điều phối container mã nguồn mở, trả lời các câu hỏi trên bằng cách cho phép bạn **khai báo trạng thái mong muốn**, rồi tự động đưa hệ thống về trạng thái đó. [Trí nhớ về mục tiêu; Đã kiểm chứng về vai trò control plane và reconciliation]

## 2. Kubernetes làm gì

| Chức năng | Ý nghĩa |
|---|---|
| Lập lịch (scheduling) | Chọn node phù hợp cho mỗi Pod |
| Tự phục hồi (self-healing) | Khởi động lại container hỏng, thay node lỗi |
| Mở rộng (scaling) | Tăng hoặc giảm số bản sao |
| Khám phá dịch vụ và cân bằng tải | Dịch vụ gọi nhau bằng tên ổn định |
| Triển khai tự động | Cập nhật phiên bản theo từng đợt, có thể quay lại |
| Quản lý cấu hình và bí mật | Tách cấu hình khỏi image |
| Quản lý lưu trữ | Gắn ổ đĩa vào container |

## 3. Phiên bản và chính sách hỗ trợ

Theo trang releases chính thức, tại thời điểm tra cứu: [Đã kiểm chứng]

| Phiên bản minor | Trạng thái | Bản vá mới nhất | Hết hỗ trợ |
|---|---|---|---|
| 1.37 | Mới nhất, được hỗ trợ | 1.37.1 (2026-09-15) | 2027-10-28 |
| 1.36 | Được hỗ trợ | 1.36.5 (2026-09-15) | 2027-06-28 |
| 1.35 | Được hỗ trợ | 1.35.9 (2026-09-15) | 2027-02-28 |
| 1.34 | Được hỗ trợ | 1.34.12 (2026-09-15) | 2026-10-27 |
| 1.33 trở xuống | Hết hỗ trợ | | |

**Chính sách:** các phiên bản từ 1.19 trở lên được hỗ trợ bản vá khoảng 1 năm. Phiên bản theo Semantic Versioning `x.y.z`. [Đã kiểm chứng]

**Khuyến nghị:** lên kế hoạch nâng cấp trước khi phiên bản hết hỗ trợ. Không chạy phiên bản đã hết hỗ trợ trong môi trường thật. [Suy luận]

---

# PHẦN B. KIẾN TRÚC CLUSTER

Cluster gồm hai nhóm: **control plane** (bộ não quyết định) và **node** (nơi chạy container).

```
┌─────────────────────── CONTROL PLANE ───────────────────────┐
│  kube-apiserver  ◄──►  etcd  (lưu trạng thái cluster)        │
│        ▲                                                     │
│        ├── kube-scheduler                  (chọn node cho Pod)│
│        ├── kube-controller-manager         (vòng lặp điều hòa)│
│        └── cloud-controller-manager        (tích hợp cloud)   │
└──────────────────────────┬──────────────────────────────────┘
                           │  (API)
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
┌──── NODE 1 ────┐  ┌──── NODE 2 ────┐  ┌──── NODE 3 ────┐
│ kubelet         │  │ kubelet         │  │ kubelet         │
│ kube-proxy      │  │ kube-proxy      │  │ kube-proxy      │
│ container runtime│ │ container runtime│ │ container runtime│
│ [Pod] [Pod]     │  │ [Pod]           │  │ [Pod] [Pod]     │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

## 4. Control plane

Theo trang kiến trúc chính thức, control plane đưa ra quyết định toàn cục và phản ứng với sự kiện trong cluster. [Đã kiểm chứng]

| Thành phần | Vai trò | Ghi chú |
|---|---|---|
| **kube-apiserver** | Cung cấp Kubernetes API, là cửa vào duy nhất của control plane | Thiết kế để scale ngang; mọi công cụ (kubectl, controller) đều đi qua nó |
| **etcd** | Kho khóa-giá trị nhất quán, lưu toàn bộ dữ liệu cluster | Cần sao lưu. Mất etcd là mất trạng thái cluster |
| **kube-scheduler** | Theo dõi Pod chưa gán node, chọn node cho từng Pod | Xem xét tài nguyên, ràng buộc, affinity, taint |
| **kube-controller-manager** | Chạy nhiều controller trong một tiến trình | Ví dụ: Node controller, Job controller, EndpointSlice controller, ServiceAccount controller |
| **cloud-controller-manager** | Tích hợp với API của nhà cung cấp cloud | Quản lý node, route, và load balancer cloud. Không có trong cluster tự dựng cục bộ |

**Lưu ý về etcd:** tài liệu khuyến nghị có kế hoạch sao lưu cho dữ liệu này. [Đã kiểm chứng]

## 5. Node components

Các thành phần chạy trên mọi node: [Đã kiểm chứng một phần]

| Thành phần | Vai trò |
|---|---|
| **kubelet** | Agent trên mỗi node, đảm bảo container chạy đúng như mô tả trong Pod |
| **kube-proxy** | Duy trì luật mạng để Service hoạt động trên mạng cluster. Một số plugin mạng tự cung cấp cơ chế này nên không cần kube-proxy |
| **Container runtime** | Chạy container, ví dụ containerd hoặc CRI-O. Kubernetes giao tiếp qua CRI (Container Runtime Interface) [Trí nhớ] |

**Lưu ý:** Docker không còn là runtime trực tiếp của kubelet từ Kubernetes 1.24. Image Docker vẫn chạy bình thường vì đây là chuẩn OCI. [Trí nhớ]

## 6. Add-on và thành phần ngoài

| Add-on | Chức năng |
|---|---|
| **CoreDNS** | DNS nội bộ cho Service và Pod |
| **Plugin mạng (CNI)** | Cấp IP và định tuyến cho Pod, ví dụ Calico, Cilium, Flannel [Trí nhớ] |
| **Metrics Server** | Cung cấp số liệu CPU và bộ nhớ cho `kubectl top` và HPA [Trí nhớ] |
| **Ingress controller hoặc Gateway** | Nhận lưu lượng HTTP từ ngoài cluster (mục 17) |
| **Dashboard, Prometheus, Grafana** | Quan sát và quản trị [Trí nhớ] |

## 7. Luồng một yêu cầu đi qua các thành phần

Ví dụ khi bạn chạy `kubectl apply -f deployment.yaml`:

```
1. kubectl gửi manifest đến kube-apiserver
2. apiserver xác thực, phân quyền, kiểm tra schema, rồi lưu vào etcd
3. Deployment controller thấy Deployment mới, tạo ReplicaSet
4. ReplicaSet controller thấy thiếu Pod, tạo Pod (chưa có node)
5. kube-scheduler thấy Pod chưa gán node, chọn node và cập nhật Pod
6. kubelet trên node đó thấy Pod được gán, gọi container runtime để chạy container
7. kubelet báo trạng thái ngược về apiserver
```

[Đã kiểm chứng về vai trò từng thành phần; các bước chi tiết là suy luận từ kiến trúc chung]

**Điểm then chốt:** các thành phần **không gọi nhau trực tiếp**. Chúng đọc và ghi trạng thái qua apiserver, nên luôn có một nguồn sự thật duy nhất là etcd. [Suy luận]

---

# PHẦN C. MÔ HÌNH ĐỐI TƯỢNG

## 8. Mô hình khai báo và vòng lặp điều hòa (reconciliation)

**Mô hình khai báo:** bạn mô tả **trạng thái mong muốn** (ví dụ "3 bản sao của ứng dụng X, image phiên bản 1.4"), không mô tả từng bước làm thế nào.

**Vòng lặp điều hòa (reconciliation loop):** mỗi controller liên tục làm:

```
lặp mãi:
    đọc trạng thái mong muốn (spec)
    đọc trạng thái thực tế (status)
    nếu khác nhau:
        thực hiện hành động để thu hẹp khoảng cách
```

**Hệ quả:** nếu một Pod chết, controller thấy có 2 bản sao thay vì 3 và tạo thêm một Pod. Bạn không cần ra lệnh tường minh. [Suy luận]

## 9. Pod

**Pod** là đơn vị nhỏ nhất Kubernetes lập lịch. Một Pod có thể chứa một hoặc nhiều container chia sẻ cùng mạng (cùng IP) và có thể chia sẻ volume. [Trí nhớ]

```yaml
apiVersion: v1                  # phiên bản API cho Pod (nhóm core)
kind: Pod                       # loại đối tượng
metadata:                       # thông tin định danh
  name: web                     # tên Pod, duy nhất trong namespace
  labels:                       # nhãn để chọn đối tượng
    app: web                    # nhãn app=web
spec:                           # trạng thái mong muốn
  containers:                   # danh sách container
    - name: nginx               # tên container trong Pod
      image: nginx:1.27         # image và phiên bản cụ thể (không dùng latest trong sản xuất)
      ports:
        - containerPort: 80     # cổng container lắng nghe (thông tin, không mở ra ngoài)
```

**Lưu ý:** Pod là **tạm thời**. Khi chết, không được khôi phục đúng nguyên trạng; controller tạo Pod mới với IP mới. Vì vậy ứng dụng không nên phụ thuộc vào IP của Pod. [Trí nhớ]

**Vòng đời Pod (phase):** `Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`. [Trí nhớ]

## 10. Label, selector, annotation

- **Label:** cặp khóa-giá trị ngắn, dùng để chọn và nhóm đối tượng.
- **Selector:** biểu thức chọn đối tượng theo label.
- **Annotation:** metadata tự do, không dùng để chọn, thường cho công cụ đọc.

```yaml
metadata:
  labels:
    app: web                    # nhãn ứng dụng
    moi-truong: production      # nhãn môi trường
  annotations:
    mo-ta: "web frontend"       # chú thích tự do, không dùng để chọn
```

```sh
kubectl get pods -l app=web                 # chọn Pod có label app=web
kubectl get pods -l 'app in (web, api)'     # chọn theo tập giá trị
```

**Nguyên tắc:** label đúng là nền để Service, Deployment, và NetworkPolicy tìm được Pod. [Suy luận]

## 11. ReplicaSet và Deployment

- **ReplicaSet:** đảm bảo đủ số Pod giống nhau đang chạy.
- **Deployment:** quản lý ReplicaSet, cung cấp cập nhật theo đợt và khả năng quay lại. Đây là đối tượng bạn thường dùng nhất cho ứng dụng không trạng thái. [Trí nhớ]

```yaml
apiVersion: apps/v1                       # Deployment thuộc nhóm apps, phiên bản v1
kind: Deployment                          # loại đối tượng
metadata:
  name: web                               # tên Deployment
spec:
  replicas: 3                             # mong muốn 3 Pod
  selector:
    matchLabels:
      app: web                            # Deployment quản lý Pod có label app=web
  strategy:
    type: RollingUpdate                   # cập nhật theo đợt (mặc định)
    rollingUpdate:
      maxSurge: 1                         # được tạo thêm tối đa 1 Pod khi cập nhật
      maxUnavailable: 0                   # không cho phép Pod nào không sẵn sàng
  template:                               # khuôn mẫu cho Pod
    metadata:
      labels:
        app: web                          # label phải khớp với selector ở trên
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
```

**Lưu ý:** `selector.matchLabels` phải khớp với label trong `template.metadata.labels`, nếu không Kubernetes từ chối manifest. [Trí nhớ]

## 12. StatefulSet, DaemonSet, Job, CronJob

| Đối tượng | Dùng cho | Đặc điểm |
|---|---|---|
| **StatefulSet** | Ứng dụng có trạng thái: cơ sở dữ liệu, hàng đợi | Pod có tên ổn định (`db-0`, `db-1`), có volume riêng |
| **DaemonSet** | Mỗi node chạy đúng một Pod | Dùng cho agent: log, giám sát, mạng |
| **Job** | Tác vụ chạy đến khi xong | Chạy một lần, có thể retry |
| **CronJob** | Tác vụ định kỳ | Theo lịch cron |

```yaml
apiVersion: batch/v1                      # Job thuộc nhóm batch
kind: Job
metadata:
  name: sao-luu
spec:
  backoffLimit: 3                         # thử lại tối đa 3 lần khi lỗi
  template:
    spec:
      restartPolicy: Never                # không tự khởi động lại container của Job
      containers:
        - name: sao-luu
          image: busybox:1.36
          command: ["sh", "-c", "echo sao lưu xong"]   # lệnh chạy một lần
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: don-dep-hang-ngay
spec:
  schedule: "0 2 * * *"                   # cron: 2 giờ sáng mỗi ngày
  jobTemplate:                            # khuôn mẫu Job tạo ra mỗi lần chạy
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: don-dep
              image: busybox:1.36
              command: ["sh", "-c", "echo dọn dẹp"]
```

**Cú pháp cron:** `phút giờ ngày-trong-tháng tháng thứ-trong-tuần`. Ví dụ `0 2 * * *` là 02:00 mỗi ngày. [Trí nhớ]

## 13. Namespace

**Namespace** chia cluster thành các vùng logic, dùng để tách môi trường hoặc nhóm (ví dụ `dev`, `staging`). Tên đối tượng là duy nhất trong namespace, không phải trong cả cluster. [Trí nhớ]

```sh
kubectl create namespace dev                 # tạo namespace dev
kubectl get pods -n dev                      # xem Pod trong namespace dev
kubectl config set-context --current --namespace=dev   # đặt namespace mặc định cho ngữ cảnh hiện tại
```

**Lưu ý:** Namespace không phải ranh giới bảo mật tuyệt đối. Cần kết hợp RBAC và NetworkPolicy. [Suy luận]

---

# PHẦN D. MẠNG VÀ KẾT NỐI

## 14. Mô hình mạng của Kubernetes

Các quy tắc cơ bản: [Trí nhớ]
- Mọi Pod có IP riêng trong mạng cluster.
- Pod nói chuyện với Pod khác mà không cần NAT (theo mô hình mạng phẳng).
- Node nói chuyện với Pod và ngược lại.

Plugin **CNI** (Container Network Interface) cài đặt các quy tắc này.

## 15. Service và các loại Service

**Service** cung cấp một địa chỉ và tên ổn định phía trước một tập Pod, vì IP của Pod thay đổi.

| Loại | Phạm vi | Mô tả |
|---|---|---|
| **ClusterIP** (mặc định) | Trong cluster | IP ảo ổn định |
| **NodePort** | Từ ngoài qua IP node | Mở một cổng trên mọi node |
| **LoadBalancer** | Từ ngoài qua cloud | Tạo load balancer của cloud (cần cloud controller) |
| **ExternalName** | Trong cluster | Ánh xạ tên sang DNS bên ngoài |
| **Headless** (`clusterIP: None`) | Trong cluster | Không có IP ảo, DNS trả về IP của từng Pod |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web                          # tên Service, cũng là tên DNS nội bộ
spec:
  type: ClusterIP                    # loại Service (mặc định nếu không ghi)
  selector:
    app: web                         # chọn các Pod có label app=web
  ports:
    - name: http                     # tên cổng
      port: 80                       # cổng Service lắng nghe
      targetPort: 80                 # cổng container nhận lưu lượng
```

```sh
kubectl get svc                    # xem danh sách Service
kubectl get endpointslices -l kubernetes.io/service-name=web   # xem Pod thật đứng sau Service
```

**Lưu ý:** Service chọn Pod theo **label**. Nếu label không khớp, Service không có endpoint và truy cập thất bại mà không báo lỗi rõ ràng. [Suy luận]

## 16. DNS nội bộ

CoreDNS cấp tên DNS cho Service theo mẫu:

```
<tên-service>.<namespace>.svc.cluster.local
```

Ví dụ `web.dev.svc.cluster.local`. Trong cùng namespace, chỉ cần `web`. [Trí nhớ]

```sh
kubectl run tam --rm -it --image=busybox:1.36 -- nslookup web   # kiểm tra DNS từ bên trong cluster
```

## 17. Ingress và Gateway API

**Ingress** định tuyến HTTP/HTTPS từ bên ngoài vào các Service theo tên miền và đường dẫn.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx            # chọn ingress controller (phải có controller tương ứng)
  rules:
    - host: web.example.com          # tên miền
      http:
        paths:
          - path: /                  # đường dẫn
            pathType: Prefix         # khớp theo tiền tố
            backend:
              service:
                name: web            # Service đích
                port:
                  number: 80
```

**Quan trọng:** tài liệu của Kubernetes thông báo ingress-nginx đã được lên lịch ngừng bảo trì: bản bảo trì nỗ lực chỉ kéo dài đến tháng 3 năm 2026, sau đó không có bản phát hành, bản vá lỗi, hay cập nhật bảo mật. [Đã kiểm chứng một phần: trích dẫn ngày từ bài công bố]

**Gateway API** là hướng thay thế được đề xuất trong các bài liên quan, với các đối tượng `Gateway` và `HTTPRoute`. Bài công bố đã đọc không nêu rõ tên thay thế; cần đối chiếu tài liệu chính thức trước khi di chuyển. [Chưa xác nhận đầy đủ]

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
    - name: my-gateway              # Gateway đứng trước
  hostnames:
    - web.example.com               # tên miền
  rules:
    - backendRefs:
        - name: web                 # Service đích
          port: 80
```

[Trí nhớ; kiểm tra phiên bản Gateway API được cài trong cluster]

## 18. NetworkPolicy

Mặc định, Pod nhận lưu lượng từ mọi nguồn. **NetworkPolicy** giới hạn lưu lượng, nhưng chỉ có tác dụng nếu plugin CNI hỗ trợ. [Trí nhớ]

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: chi-cho-phep-tu-api
spec:
  podSelector:                       # áp dụng cho Pod có label này
    matchLabels:
      app: db
  policyTypes:
    - Ingress                        # quy tắc áp cho lưu lượng vào
  ingress:
    - from:
        - podSelector:               # chỉ cho phép từ Pod có label này
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432                 # chỉ cổng 5432
```

**Nguyên tắc:** bắt đầu bằng chính sách mặc định "từ chối tất cả" rồi mở dần. [Suy luận]

---

# PHẦN E. CẤU HÌNH VÀ LƯU TRỮ

## 19. ConfigMap và Secret

- **ConfigMap:** cấu hình không bí mật.
- **Secret:** dữ liệu nhạy cảm (mật khẩu, token). Dữ liệu được mã hóa base64, **không phải mã hóa**. Cần bật mã hóa lúc nghỉ (encryption at rest) và kiểm soát RBAC. [Trí nhớ]

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cau-hinh-web
data:
  MUC_LOG: "info"                    # cặp khóa-giá trị, giá trị là chuỗi
  CHE_DO: "production"
---
apiVersion: v1
kind: Secret
metadata:
  name: bi-mat-db
type: Opaque                         # loại Secret thông thường
stringData:                          # nhập văn bản thường, Kubernetes tự mã hóa base64
  MAT_KHAU: "thay-doi-khi-dung"
```

**Dùng trong Pod:**

```yaml
spec:
  containers:
    - name: web
      image: nginx:1.27
      envFrom:
        - configMapRef:
            name: cau-hinh-web   # đưa mọi khóa của ConfigMap thành biến môi trường
        - secretRef:
            name: bi-mat-db      # đưa mọi khóa của Secret thành biến môi trường
```

```sh
kubectl create configmap cau-hinh --from-file=config.txt          # tạo từ tệp
kubectl create secret generic bi-mat --from-literal=MAT_KHAU=abc  # tạo Secret từ chuỗi
```

**Lưu ý:** Secret qua biến môi trường có thể lộ qua log hoặc `kubectl describe`. Ưu tiên mount dưới dạng tệp khi có thể. [Trí nhớ]

## 20. Volume, PersistentVolume, PersistentVolumeClaim, StorageClass

Container mất dữ liệu khi khởi động lại. **Volume** giữ dữ liệu lâu hơn.

| Khái niệm | Vai trò |
|---|---|
| **Volume** | Gắn lưu trữ vào Pod, vòng đời gắn với Pod |
| **PersistentVolume (PV)** | Tài nguyên lưu trữ trong cluster, do quản trị viên hoặc tự động tạo |
| **PersistentVolumeClaim (PVC)** | Yêu cầu lưu trữ của ứng dụng |
| **StorageClass** | Mẫu tạo PV tự động, ví dụ loại ổ đĩa cloud |

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: du-lieu-db
spec:
  accessModes:
    - ReadWriteOnce                  # một node đọc-ghi được
  storageClassName: standard         # tên StorageClass (kiểm tra cluster có class này)
  resources:
    requests:
      storage: 10Gi                  # yêu cầu 10 GiB
```

```yaml
# Dùng trong Pod
spec:
  containers:
    - name: db
      image: postgres:16
      volumeMounts:
        - name: du-lieu
          mountPath: /var/lib/postgresql/data   # nơi gắn trong container
  volumes:
    - name: du-lieu
      persistentVolumeClaim:
        claimName: du-lieu-db        # dùng PVC đã tạo
```

**Chế độ truy cập:** `ReadWriteOnce` (một node), `ReadOnlyMany` (nhiều node chỉ đọc), `ReadWriteMany` (nhiều node đọc-ghi, tùy hỗ trợ của storage). [Trí nhớ]

---

# PHẦN F. LẬP LỊCH, TÀI NGUYÊN, VÀ TỰ CO GIÃN

## 21. Requests và limits

```yaml
resources:
  requests:                          # tài nguyên tối thiểu, dùng để lập lịch
    cpu: "250m"                      # 250 millicore = 0,25 CPU
    memory: "256Mi"                  # 256 mebibyte
  limits:                            # tài nguyên tối đa
    cpu: "500m"                      # vượt quá sẽ bị giới hạn (throttle)
    memory: "512Mi"                  # vượt quá sẽ bị OOMKill
```

**Ý nghĩa:**
- `requests` quyết định scheduler đặt Pod ở node nào.
- `limits` giới hạn tài nguyên thực tế; bộ nhớ vượt limit làm container bị kill (OOMKilled). [Trí nhớ]

**Khuyến nghị:** luôn đặt `requests` và `limits` để tránh một Pod chiếm hết node. [Suy luận]

## 22. Probes: liveness, readiness, startup

| Probe | Câu hỏi | Nếu thất bại |
|---|---|---|
| **liveness** | Container còn sống không? | Kubelet khởi động lại container |
| **readiness** | Container sẵn sàng nhận lưu lượng không? | Gỡ Pod khỏi endpoint của Service |
| **startup** | Ứng dụng đã khởi động xong chưa? | Trì hoãn liveness cho đến khi xong |

```yaml
containers:
  - name: web
    image: nginx:1.27
    readinessProbe:
      httpGet:                       # kiểm tra bằng HTTP GET
        path: /healthz               # đường dẫn kiểm tra
        port: 80                     # cổng
      initialDelaySeconds: 5         # chờ 5 giây trước lần kiểm tra đầu
      periodSeconds: 10              # kiểm tra mỗi 10 giây
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
```

**Lưu ý:** liveness quá nhạy có thể khởi động lại container liên tục. Readiness thường là lựa chọn an toàn hơn. [Suy luận]

## 23. Lập lịch: nodeSelector, affinity, taint, toleration

```yaml
spec:
  nodeSelector:                      # chỉ đặt lên node có label này
    disktype: ssd
  affinity:
    nodeAffinity:                    # quy tắc node phức tạp hơn
      requiredDuringSchedulingIgnoredDuringExecution:   # bắt buộc khi lập lịch
        nodeSelectorTerms:
          - matchExpressions:
              - key: zone
                operator: In        # giá trị nằm trong danh sách
                values: ["hanoi-a"]
    podAntiAffinity:                 # tránh đặt chung node với Pod khác
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: web
            topologyKey: kubernetes.io/hostname
  tolerations:                       # chấp nhận node có taint
    - key: "dac-biet"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

**Taint và toleration:**
- **Taint** đặt trên node, đẩy Pod ra xa trừ khi Pod có toleration phù hợp.
- **Toleration** đặt trên Pod, cho phép Pod đến node có taint. [Trí nhớ]

```sh
kubectl taint nodes node1 dac-biet=true:NoSchedule      # đặt taint
kubectl taint nodes node1 dac-biet=true:NoSchedule-     # xóa taint (dấu - ở cuối)
```

## 24. Tự co giãn: HPA, VPA, PDB

**HPA (Horizontal Pod Autoscaler):** tăng hoặc giảm số bản sao theo số liệu (thường là CPU). Cần Metrics Server. [Trí nhớ]

```yaml
apiVersion: autoscaling/v2           # phiên bản API HPA
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:                    # đối tượng cần co giãn
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2                     # tối thiểu 2 Pod
  maxReplicas: 10                    # tối đa 10 Pod
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70     # mục tiêu: trung bình 70% CPU request
```

**VPA (Vertical Pod Autoscaler):** điều chỉnh requests và limits của Pod. Thường cần add-on riêng. [Trí nhớ]

**PDB (PodDisruptionBudget):** giới hạn số Pod có thể bị gián đoạn cùng lúc khi bảo trì node. [Trí nhớ]

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2                    # luôn giữ ít nhất 2 Pod chạy
  selector:
    matchLabels:
      app: web
```

---

# PHẦN G. BẢO MẬT

## 25. Xác thực, phân quyền, RBAC

- **Xác thực (authentication):** apiserver nhận diện người gọi qua chứng chỉ, token, hoặc OIDC.
- **Phân quyền (authorization):** RBAC (Role-Based Access Control) là cơ chế phổ biến. [Trí nhớ]

**Các đối tượng RBAC:**

| Đối tượng | Phạm vi | Vai trò |
|---|---|---|
| **Role** | Một namespace | Tập quyền |
| **ClusterRole** | Toàn cluster | Tập quyền |
| **RoleBinding** | Một namespace | Gắn Role cho người dùng hoặc ServiceAccount |
| **ClusterRoleBinding** | Toàn cluster | Gắn ClusterRole |

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: doc-pod
  namespace: dev
rules:
  - apiGroups: [""]                  # nhóm core (rỗng)
    resources: ["pods"]              # tài nguyên được áp dụng
    verbs: ["get", "list", "watch"]  # hành động được phép (chỉ đọc)
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: doc-pod-cho-nhan-vien
  namespace: dev
subjects:
  - kind: User
    name: nguoi-dung-a              # người dùng được cấp quyền
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: doc-pod                     # Role được gắn
  apiGroup: rbac.authorization.k8s.io
```

```sh
kubectl auth can-i list pods -n dev                       # tôi có quyền liệt kê Pod không?
kubectl auth can-i delete pods -n dev --as nguoi-dung-a   # kiểm tra quyền của người khác
```

**Nguyên tắc:** cấp quyền tối thiểu, tránh `cluster-admin` cho ứng dụng. [Suy luận]

## 26. ServiceAccount

ServiceAccount là danh tính của tiến trình trong Pod, khi Pod gọi apiserver. [Trí nhớ]

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: web-sa
---
spec:
  serviceAccountName: web-sa         # Pod dùng ServiceAccount này
  automountServiceAccountToken: false  # không tự gắn token nếu ứng dụng không gọi apiserver
```

**Lưu ý:** tắt tự động gắn token khi ứng dụng không cần gọi API Kubernetes, để giảm rủi ro. [Trí nhớ]

## 27. Pod Security và bảo mật container

**Pod Security Standards** định nghĩa ba mức: `privileged`, `baseline`, `restricted`. Áp dụng qua label của namespace. [Trí nhớ]

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    pod-security.kubernetes.io/enforce: restricted   # áp mức restricted cho namespace này
```

Thiết lập an toàn cho container:

```yaml
securityContext:
  runAsNonRoot: true                 # không chạy bằng root
  allowPrivilegeEscalation: false    # không cho leo quyền
  readOnlyRootFilesystem: true       # hệ tệp gốc chỉ đọc
  capabilities:
    drop: ["ALL"]                    # bỏ mọi capability Linux
```

[Trí nhớ; tên trường và mức cần kiểm tra theo phiên bản]

---

# PHẦN H. WORKFLOW

## 28. Workflow phát triển và triển khai

```
1. Viết mã và Dockerfile
2. Build image, gắn tag cụ thể (không dùng latest)
3. Đẩy image lên registry
4. Viết manifest (Deployment, Service, ...) hoặc Helm chart
5. Thử trên cluster cục bộ (kind, minikube)
6. Kiểm tra: kubectl apply --dry-run=server, kubectl diff
7. Áp dụng lên môi trường: kubectl apply hoặc helm upgrade
8. Theo dõi rollout: kubectl rollout status
9. Kiểm tra log, metric, và endpoint
10. Quay lại nếu lỗi: kubectl rollout undo
```

```sh
docker build -t registry.example.com/web:1.4.2 .        # build image với tag rõ ràng
docker push registry.example.com/web:1.4.2             # đẩy lên registry
kubectl apply --dry-run=server -f deployment.yaml      # kiểm tra phía server, không thay đổi thật
kubectl diff -f deployment.yaml                        # xem khác biệt so với trạng thái hiện tại
kubectl apply -f deployment.yaml                       # áp dụng
kubectl rollout status deployment/web                  # theo dõi cập nhật
```

## 29. Rolling update, rollback, và chiến lược triển khai

| Chiến lược | Cách hoạt động | Ghi chú |
|---|---|---|
| **RollingUpdate** (mặc định) | Thay dần Pod cũ bằng Pod mới | Không ngừng dịch vụ nếu có readiness |
| **Recreate** | Dừng toàn bộ Pod cũ, rồi tạo mới | Có downtime, đơn giản |
| **Blue/Green** | Hai môi trường song song, chuyển lưu lượng | Cần công cụ hoặc Service/Ingress hỗ trợ [Trí nhớ] |
| **Canary** | Đưa một phần nhỏ lưu lượng sang phiên bản mới | Cần công cụ như Argo Rollouts hoặc service mesh [Trí nhớ] |

```sh
kubectl rollout history deployment/web             # xem lịch sử cập nhật
kubectl rollout undo deployment/web                # quay về phiên bản trước
kubectl rollout undo deployment/web --to-revision=2   # quay về revision cụ thể
```

## 30. Helm và Kustomize

**Helm** là trình quản lý gói cho Kubernetes: đóng gói manifest thành chart, có tham số hóa, và quản lý phiên bản release. [Trí nhớ]

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami    # thêm kho chart
helm search repo nginx                                       # tìm chart
helm install web bitnami/nginx -n dev --create-namespace      # cài release tên web
helm upgrade web bitnami/nginx -n dev --set replicaCount=3   # nâng cấp với tham số
helm rollback web 1 -n dev                                   # quay về revision 1
helm uninstall web -n dev                                    # gỡ release
helm template web bitnami/nginx > rendered.yaml              # xem manifest sinh ra, không cài
```

[Tên repo và chart là minh họa; kiểm tra tài liệu của chart cụ thể]

**Kustomize** tích hợp sẵn trong `kubectl`, dùng overlay để tùy biến manifest theo môi trường mà không nhân bản file. [Trí nhớ]

```sh
kubectl apply -k overlays/production        # áp dụng bản đã tùy biến cho production
kubectl kustomize overlays/production       # xem kết quả trước khi áp dụng
```

## 31. GitOps và CI/CD

**GitOps:** Git là nguồn sự thật cho trạng thái cluster. Một công cụ như Argo CD hoặc Flux đồng bộ cluster với repo. [Trí nhớ]

Pipeline CI/CD điển hình:

```
1. Commit mã → CI chạy test và lint
2. CI build image, quét lỗ hổng, đẩy registry
3. CI cập nhật tag image trong repo manifest (hoặc Helm values)
4. Công cụ GitOps (hoặc CD) áp dụng thay đổi lên cluster
5. Kiểm tra rollout và rollback tự động khi lỗi
```

**Ví dụ GitHub Actions** (minh họa):

```yaml
name: deploy
on:
  push:
    branches: [main]                 # chạy khi push lên nhánh main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4    # tải mã
      - run: echo "build và push image ở đây"   # thay bằng các bước build thật
      - run: kubectl apply -f k8s/   # áp dụng manifest (cần cấu hình kubeconfig an toàn)
```

**Lưu ý:** không lưu kubeconfig có quyền quản trị trong mã nguồn. Dùng secret của CI và quyền tối thiểu. [Suy luận]

---

# PHẦN I. CÂU LỆNH KUBECTL

`kubectl` là công cụ dòng lệnh chính để làm việc với cluster. Cú pháp chung: `kubectl <động-từ> <loại> <tên> [cờ]`.

## 32. Cấu hình và ngữ cảnh

```sh
kubectl version                              # xem phiên bản client và server
kubectl cluster-info                         # xem địa chỉ control plane
kubectl config view                          # xem cấu hình kubeconfig
kubectl config get-contexts                  # liệt kê ngữ cảnh (cluster và người dùng)
kubectl config current-context               # xem ngữ cảnh hiện tại
kubectl config use-context kind-kind         # chuyển sang ngữ cảnh khác
kubectl config set-context --current --namespace=dev   # đặt namespace mặc định
```

## 33. Xem tài nguyên

```sh
kubectl get pods                             # liệt kê Pod trong namespace hiện tại
kubectl get pods -A                          # mọi namespace (-A = --all-namespaces)
kubectl get pods -o wide                     # thêm cột node và IP
kubectl get pods -o yaml                     # xuất đầy đủ dạng YAML
kubectl get pods -o jsonpath='{.items[*].metadata.name}'   # lọc bằng JSONPath
kubectl get pods -l app=web                  # lọc theo label
kubectl get pods --field-selector status.phase=Running     # lọc theo trường
kubectl get pods -w                          # theo dõi thay đổi (watch)
kubectl get deploy,svc,ing                   # nhiều loại cùng lúc
kubectl get all                              # hầu hết tài nguyên trong namespace (không đầy đủ, chỉ là gợi ý)
kubectl describe pod web-7d9c8-abcde         # chi tiết: sự kiện, trạng thái, điều kiện
kubectl api-resources                        # liệt kê loại tài nguyên và viết tắt
kubectl explain deployment.spec.strategy     # giải thích trường trong schema
kubectl top pods                             # CPU và bộ nhớ (cần Metrics Server)
kubectl top nodes                            # số liệu node
kubectl get events --sort-by=.lastTimestamp  # sự kiện theo thời gian (gỡ lỗi)
```

## 34. Tạo, sửa, xóa

```sh
kubectl apply -f deployment.yaml             # tạo hoặc cập nhật theo file (khuyến nghị)
kubectl apply -f thu-muc/                    # áp dụng mọi file trong thư mục
kubectl apply -k overlays/dev                # áp dụng bản Kustomize
kubectl create deployment web --image=nginx:1.27   # tạo Deployment nhanh (không khai báo file)
kubectl create namespace dev                 # tạo namespace
kubectl apply -f deployment.yaml --dry-run=server  # kiểm tra phía server, không ghi
kubectl diff -f deployment.yaml              # so sánh với trạng thái hiện tại
kubectl edit deployment web                  # sửa trực tiếp trong trình soạn thảo
kubectl patch deployment web -p '{"spec":{"replicas":5}}'   # sửa một phần bằng JSON
kubectl label pod web-abc app=web-v2         # thêm hoặc sửa label
kubectl annotate pod web-abc owner=team-a    # thêm annotation
kubectl delete pod web-abc                   # xóa Pod (Deployment sẽ tạo Pod mới)
kubectl delete -f deployment.yaml            # xóa theo file
kubectl delete pods -l app=web               # xóa theo label
kubectl delete pod web-abc --grace-period=0 --force   # xóa ngay (thận trọng)
```

**Lưu ý:** `apply` ghi nhớ trạng thái đã áp dụng (qua annotation `last-applied`). `create` thất bại nếu đã tồn tại. Chọn một cách và dùng nhất quán. [Trí nhớ]

## 35. Chạy và gỡ lỗi

```sh
kubectl run tam --image=busybox:1.36 --rm -it --restart=Never -- sh   # chạy Pod tạm để thử
kubectl exec -it web-abc -- sh               # mở shell trong container
kubectl exec web-abc -- ls /app              # chạy lệnh một lần
kubectl logs web-abc                         # xem log container
kubectl logs web-abc -c nginx                # chọn container khi Pod có nhiều container
kubectl logs web-abc --previous              # log của container lần chạy trước (khi bị restart)
kubectl logs -f web-abc                      # theo dõi log trực tiếp (follow)
kubectl logs -l app=web --all-containers     # log của mọi Pod có label
kubectl port-forward pod/web-abc 8080:80     # chuyển tiếp cổng máy 8080 đến cổng 80 của Pod
kubectl port-forward svc/web 8080:80         # chuyển tiếp qua Service
kubectl cp web-abc:/app/log.txt ./log.txt    # sao chép tệp từ Pod về máy
kubectl cp ./config.txt web-abc:/app/        # sao chép tệp lên Pod
kubectl debug -it web-abc --image=busybox:1.36 --target=nginx   # gắn container gỡ lỗi vào Pod
```

## 36. Quản lý node

```sh
kubectl get nodes                            # liệt kê node
kubectl describe node node1                  # chi tiết node: tài nguyên, điều kiện, taint
kubectl cordon node1                         # đánh dấu không lập lịch Pod mới lên node
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data   # sơ tán Pod khỏi node để bảo trì
kubectl uncordon node1                       # cho phép lập lịch trở lại
kubectl taint nodes node1 key=value:NoSchedule   # đặt taint
```

## 37. Rollout và scale

```sh
kubectl scale deployment web --replicas=5    # đặt số bản sao
kubectl autoscale deployment web --min=2 --max=10 --cpu-percent=70   # tạo HPA nhanh
kubectl set image deployment/web nginx=nginx:1.28   # đổi image (kích hoạt rollout)
kubectl rollout status deployment/web        # theo dõi cập nhật
kubectl rollout history deployment/web       # lịch sử
kubectl rollout undo deployment/web          # quay lại bản trước
kubectl rollout restart deployment/web       # khởi động lại mọi Pod (ví dụ khi đổi ConfigMap)
kubectl rollout pause deployment/web         # tạm dừng rollout
kubectl rollout resume deployment/web        # tiếp tục rollout
```

## 38. Mạng và truy cập

```sh
kubectl expose deployment web --port=80 --target-port=80 --type=ClusterIP   # tạo Service nhanh
kubectl get svc                              # liệt kê Service
kubectl get ingress                          # liệt kê Ingress
kubectl get endpointslices                   # xem Pod đứng sau Service
kubectl get networkpolicy                    # liệt kê NetworkPolicy
kubectl proxy                                # mở proxy tới API server (cổng 8001)
kubectl auth can-i create deployments        # kiểm tra quyền
kubectl auth whoami                          # xem danh tính hiện tại (phiên bản hỗ trợ) [Trí nhớ]
```

## 39. Lệnh Helm thường dùng

| Lệnh | Tác dụng |
|---|---|
| `helm repo add <tên> <url>` | Thêm kho chart |
| `helm repo update` | Cập nhật danh mục chart |
| `helm search repo <từ-khóa>` | Tìm chart |
| `helm install <release> <chart> -n <ns>` | Cài chart |
| `helm upgrade <release> <chart>` | Nâng cấp |
| `helm upgrade --install ...` | Cài nếu chưa có, nâng cấp nếu đã có (dùng trong CI) |
| `helm list -A` | Liệt kê release mọi namespace |
| `helm history <release>` | Lịch sử revision |
| `helm rollback <release> <revision>` | Quay về revision |
| `helm uninstall <release>` | Gỡ |
| `helm template <release> <chart>` | Xem manifest sinh ra |
| `helm lint <thư-mục-chart>` | Kiểm tra chart |

---

# PHẦN J. GỠ LỖI

## 40. Quy trình gỡ lỗi

Đi từ tổng quan đến chi tiết:

```
1. Cái gì không chạy?       kubectl get pods, deploy, svc, ing
2. Pod ở trạng thái nào?    kubectl get pods -o wide (xem STATUS, RESTARTS, NODE)
3. Vì sao?                  kubectl describe pod <tên>   (đọc phần Events ở cuối)
4. Log nói gì?              kubectl logs <tên> [--previous]
5. Có kết nối không?        kubectl get endpointslices; kubectl exec ... -- curl ...
6. Có sự kiện cluster không? kubectl get events --sort-by=.lastTimestamp
7. Cấu hình đúng không?     kubectl describe, kubectl diff, kubectl get -o yaml
```

## 41. Các trạng thái Pod thường gặp

| Trạng thái | Ý nghĩa | Hướng xử lý |
|---|---|---|
| `Pending` | Chưa được lập lịch | `describe` xem thiếu tài nguyên, taint, hoặc PVC chưa bind |
| `ImagePullBackOff` / `ErrImagePull` | Không kéo được image | Kiểm tra tên image, tag, quyền registry |
| `CrashLoopBackOff` | Container chết liên tục | `logs --previous`, xem lỗi khởi động |
| `OOMKilled` | Vượt giới hạn bộ nhớ | Tăng limit hoặc tối ưu ứng dụng |
| `CreateContainerConfigError` | Thiếu ConfigMap hoặc Secret | Tạo đối tượng còn thiếu |
| `Running` nhưng không sẵn sàng | Readiness thất bại | Kiểm tra probe và endpoint của ứng dụng |
| `Terminating` kéo dài | Pod không thoát | Kiểm tra finalizer hoặc node không phản hồi |

```sh
kubectl describe pod web-abc | tail -n 20      # xem phần Events cuối cùng
```

---

## 42. Gỡ lỗi theo lớp: từ triệu chứng đến nguyên nhân

Đây là phần mở rộng của mục 40–41. Nguyên tắc: **xác định triệu chứng, khoanh vùng lớp (Pod, Service, Ingress, DNS, node, control plane), rồi kiểm tra từng lớp từ dưới lên hoặc từ ngoài vào**. Không sửa trực tiếp trên môi trường sản xuất khi chưa có bằng chứng.

### 42.1. Năm câu hỏi đầu tiên

| # | Câu hỏi | Lệnh gợi ý |
|---|---|---|
| 1 | Lỗi xảy ra từ khi nào? Có thay đổi gì ngay trước đó không? | `kubectl rollout history`, `helm history`, lịch sử CI/CD |
| 2 | Lỗi ảnh hưởng một Pod, một dịch vụ, hay toàn cluster? | `kubectl get pods -A`, `kubectl get nodes` |
| 3 | Có sự kiện nào trong cluster không? | `kubectl get events -A --sort-by=.lastTimestamp` |
| 4 | Ứng dụng báo lỗi gì? | `kubectl logs`, `kubectl logs --previous` |
| 5 | Tài nguyên có đủ không? | `kubectl top pods`, `kubectl top nodes`, `kubectl describe node` |

**Ghi chú:** `kubectl get events` mặc định chỉ giữ một khoảng thời gian ngắn. Nếu sự cố đã lâu, cần log tập trung hoặc hệ thống giám sát. [Trí nhớ]

### 42.2. Bản đồ triệu chứng → lớp → lệnh

| Triệu chứng | Lớp nghi ngờ | Lệnh kiểm tra đầu tiên |
|---|---|---|
| Pod không chạy (`Pending`) | Lập lịch, tài nguyên, PVC | `kubectl describe pod <tên>` |
| Pod chạy nhưng crash | Ứng dụng, cấu hình, probe, bộ nhớ | `kubectl logs <tên> --previous`, `describe` |
| Pod `Running` nhưng `0/1 READY` | Readiness probe | `describe` (phần Conditions và Events) |
| Service không truy cập được | Selector, endpoint, targetPort | `kubectl get endpointslices -l kubernetes.io/service-name=<svc>` |
| Tên DNS không phân giải | CoreDNS, namespace, ndots | `nslookup` từ trong Pod |
| Ingress trả 404, 502, 503 | Ingress class, host/path, backend | `kubectl describe ingress`, log của ingress controller |
| Pod kẹt `Terminating` | Finalizer, webhook, node mất kết nối | `describe`, `get pod -o yaml` (mục `finalizers`) |
| `Forbidden` khi chạy lệnh | RBAC | `kubectl auth can-i --as <user>` |
| `NotReady` ở node | kubelet, tài nguyên node, mạng | `kubectl describe node <tên>` |
| `kubectl` không kết nối được | Kubeconfig, apiserver, mạng | `kubectl config view`, `kubectl cluster-info` |
| Cập nhật không chạy | Rollout, image, tài nguyên | `kubectl rollout status`, `describe deployment` |

### 42.3. Pod ở trạng thái Pending

**Nguyên nhân thường gặp** (đọc dòng `FailedScheduling` trong phần Events): [Đã kiểm chứng một phần: trang Debug Pods liệt kê Pending là do không lập lịch được, kiểm tra bằng `describe` và có nguyên nhân thiếu CPU/bộ nhớ hoặc dùng `hostPort`]

| Thông báo (ví dụ) | Ý nghĩa | Hướng xử lý |
|---|---|---|
| `Insufficient cpu` / `Insufficient memory` | Không node nào đủ `requests` | Giảm `requests`, xóa Pod thừa, thêm node |
| `node(s) had untolerated taint` | Node có taint, Pod không có toleration | Thêm toleration hoặc gỡ taint |
| `didn't match Pod's node affinity/selector` | `nodeSelector` hoặc affinity không khớp node nào | Kiểm tra label node và selector |
| `pod has unbound immediate PersistentVolumeClaims` | PVC chưa được cấp PV | Xem mục 42.9 |
| `node(s) didn't have free ports for the requested pod ports` | `hostPort` trùng | Dùng Service thay vì `hostPort` |
| `didn't match pod anti-affinity rules` | Anti-affinity không thỏa | Nới quy tắc hoặc thêm node |

```sh
kubectl describe pod <tên> | grep -A 20 Events          # đọc sự kiện lập lịch
kubectl get nodes --show-labels                        # xem label node để so với selector
kubectl describe node <tên> | grep -A 10 Taints         # xem taint của node
kubectl get nodes -o custom-columns=NAME:.metadata.name,CPU:.status.allocatable.cpu,MEM:.status.allocatable.memory
```

**Lưu ý:** Pod `Pending` do thiếu tài nguyên thường không tự hết. Phải thay đổi requests, node, hoặc quota. [Suy luận]

### 42.4. Pod không khởi động: image và container

**Image không kéo được** (`ErrImagePull`, `ImagePullBackOff`):
- Kiểm tra tên image và tag có tồn tại trong registry không.
- Kiểm tra secret kéo image (`imagePullSecrets`) và quyền truy cập.
- Thử kéo thủ công trên node hoặc máy có quyền: `docker pull <image>`. Đây là lệnh của Docker, không phải kubectl. [Đã kiểm chứng: trang Debug Pods gợi ý cách này]

**Container crash** (`CrashLoopBackOff`): Kubernetes khởi động lại container nhiều lần và tăng dần thời gian chờ giữa các lần.

```sh
kubectl logs <tên> --previous             # log của lần chạy trước khi crash (quan trọng nhất)
kubectl logs <tên> -c <container>         # chọn container khi Pod có nhiều container
kubectl describe pod <tên>                # xem Last State và Exit Code
kubectl get pod <tên> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.exitCode}'
```

**Bảng mã thoát thường gặp:** [Trí nhớ; mã thoát là quy ước của Unix]

| Exit code | Ý nghĩa thường gặp | Hướng xử lý |
|---|---|---|
| 0 | Chạy xong bình thường | Nếu là service, có thể lệnh kết thúc sớm |
| 1 | Lỗi ứng dụng chung | Đọc log |
| 126 | Không có quyền thực thi lệnh | Kiểm tra quyền file, `command` |
| 127 | Không tìm thấy lệnh | Sai `command` hoặc thiếu binary trong image |
| 137 | Bị SIGKILL, thường do OOM hoặc bị kill | Kiểm tra `OOMKilled`, tăng limit |
| 139 | Lỗi truy cập bộ nhớ (segfault) | Lỗi trong ứng dụng hoặc thư viện |
| 143 | Nhận SIGTERM và thoát | Bình thường khi bị dừng; kiểm tra xử lý tín hiệu |

**Các nguyên nhân khác của CrashLoopBackOff:**
- **Liveness probe quá nghiêm:** kubelet giết container đang khởi động chậm. Dùng `startupProbe` hoặc tăng `initialDelaySeconds`.
- **Thiếu biến môi trường hoặc ConfigMap/Secret:** ứng dụng thoát khi đọc cấu hình. Kiểm tra `describe` và log.
- **Sai `command` hoặc `args`:** lệnh không tồn tại trong image.
- **Quyền hệ thống tệp:** `readOnlyRootFilesystem: true` nhưng ứng dụng cố ghi vào thư mục gốc.

**Thử container trong Pod đang lỗi bằng container gỡ lỗi:**

```sh
kubectl debug -it <tên-pod> --image=busybox:1.36 --target=<container>   # gắn container tạm vào Pod (ephemeral)
kubectl debug <tên-pod> -it --copy-to=<tên-pod>-debug --container=<c> -- sh   # tạo bản sao Pod để thử
```

[Trí nhớ; cú pháp `kubectl debug` cần kiểm tra theo phiên bản]

### 42.5. Pod chạy nhưng không sẵn sàng (readiness)

```sh
kubectl get pods -l app=<nhãn>                           # xem cột READY, ví dụ 0/1
kubectl describe pod <tên> | grep -A 5 Readiness         # xem probe đang kiểm tra gì
kubectl exec <tên> -- curl -sv localhost:<cổng>/healthz  # tự kiểm tra endpoint từ bên trong
```

**Các nguyên nhân thường gặp:**
- Đường dẫn probe sai (`/healthz` không tồn tại, trả 404).
- Cổng probe khác cổng ứng dụng lắng nghe.
- Ứng dụng chỉ lắng nghe `127.0.0.1` thay vì `0.0.0.0`, nên probe từ kubelet không kết nối được. [Suy luận]
- Ứng dụng phụ thuộc dịch vụ khác (cơ sở dữ liệu) chưa sẵn sàng, nên readiness luôn thất bại.

Khi readiness thất bại, Pod bị gỡ khỏi endpoint của Service, nên Service có vẻ "không có backend" dù Pod đang chạy. [Suy luận]

### 42.6. Pod kẹt ở `Terminating`

**Nguyên nhân:** có finalizer trên Pod mà controller không gỡ được, hoặc webhook chặn thao tác cập nhật. [Đã kiểm chứng: trang Debug Pods nêu finalizer và webhook là nguyên nhân]

```sh
kubectl get pod <tên> -o yaml | grep -A 3 finalizers               # xem finalizer
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations   # kiểm tra webhook
kubectl get pod <tên> -o wide                                      # node có còn sống không
```

**Biện pháp (thận trọng):**
- Với webhook của bên thứ ba: cập nhật, tắt cho thao tác `UPDATE`, hoặc báo nhà cung cấp.
- Với webhook tự viết: webhook dạng mutating không được đổi trường bất biến khi `UPDATE`. Webhook dạng validating chỉ nên áp dụng cho thay đổi mới, để Pod đang chạy không bị kẹt. [Đã kiểm chứng]
- Xóa cưỡng bức chỉ dùng khi đã hiểu hậu quả: `kubectl delete pod <tên> --grace-period=0 --force`. Nếu node đã mất, phải xử lý node. [Suy luận]

### 42.7. Cấu hình không có hiệu lực (lỗi cú pháp im lặng)

Có trường hợp Pod chạy bình thường nhưng không làm đúng ý, vì một khóa bị sai chính tả hoặc sai cấp và bị bỏ qua. Ví dụ `commnd` thay vì `command`. [Đã kiểm chứng: trang Debug Pods]

```sh
kubectl apply --dry-run=server -f pod.yaml        # báo lỗi trường không hợp lệ
kubectl apply --validate=true -f pod.yaml         # kiểm tra schema phía client
kubectl get pod <tên> -o yaml > tren-apiserver.yaml   # xuất trạng thái thật trên apiserver
diff pod.yaml tren-apiserver.yaml                 # so sánh (dòng thừa phía apiserver là bình thường)
kubectl explain pod.spec.containers               # kiểm tra tên trường hợp lệ
```

**Quy tắc:** nếu dòng có trong file gốc nhưng biến mất trong bản trên apiserver, nhiều khả năng khóa đó sai và đã bị bỏ qua. [Đã kiểm chứng]

### 42.8. Service không truy cập được

Đi theo thứ tự, theo tài liệu Debug Services: [Đã kiểm chứng một phần]

```sh
# 1. Service có tồn tại không?
kubectl get svc <svc>                                 # NotFound nghĩa là chưa tạo

# 2. Selector có khớp label của Pod không?
kubectl get svc <svc> -o jsonpath='{.spec.selector}{"\n"}'
kubectl get pods --show-labels                        # so sánh label

# 3. Có endpoint không?
kubectl get endpointslices -l kubernetes.io/service-name=<svc>
kubectl get pods -l <selector> -o wide                # Pod có IP và READY chưa

# 4. Pod có trả lời trực tiếp không? (chạy từ Pod gỡ lỗi)
kubectl run tam --rm -it --restart=Never --image=busybox:1.36 -- sh
wget -qO- http://<pod-ip>:<cổng>                      # kiểm tra từ bên trong cluster

# 5. port và targetPort có đúng không?
kubectl get svc <svc> -o yaml | grep -E 'port|targetPort'
```

**Các lỗi thường gặp:**
- **Selector không khớp label:** Service không có endpoint. Đây là lỗi phổ biến nhất.
- **`targetPort` không khớp cổng container lắng nghe.** Pod đang sẵn sàng nhưng kết nối bị từ chối.
- **Pod không `READY`:** Service không đưa Pod vào endpoint.
- **NetworkPolicy chặn lưu lượng:** kiểm tra `kubectl get networkpolicy -A`. [Đã kiểm chứng: trang Debug Services nhắc đến rà soát chính sách Ingress]

**Kiểm tra kube-proxy** khi ClusterIP không hoạt động nhưng IP của Pod thì có: [Đã kiểm chứng một phần]

```sh
kubectl get pods -n kube-system -l k8s-app=kube-proxy   # nhãn có thể khác theo cluster
kubectl logs -n kube-system <kube-proxy-pod>             # tìm lỗi quy tắc mạng
```

### 42.9. Lưu trữ: PVC không được gắn

```sh
kubectl get pvc                                         # trạng thái Pending hoặc Bound
kubectl describe pvc <tên>                              # đọc sự kiện: provisioner lỗi, StorageClass sai
kubectl get storageclass                                # StorageClass có tồn tại và có mặc định không
kubectl get pv                                          # PV có sẵn để gắn không
kubectl describe pod <tên> | grep -A 5 Volumes          # Pod có tham chiếu đúng PVC không
```

**Nguyên nhân thường gặp:**
- Tên `storageClassName` sai, hoặc không có StorageClass mặc định.
- Provisioner (CSI driver) chưa cài hoặc lỗi.
- Chế độ truy cập không khớp: `ReadWriteMany` yêu cầu storage hỗ trợ nhiều node.
- Volume đang gắn vào node khác (lỗi attach), thường gặp khi Pod chuyển node.
- Zone không khớp: volume tạo ở zone A không gắn được vào node ở zone B. [Trí nhớ]

### 42.10. Ingress trả lỗi (404, 502, 503)

```sh
kubectl get ingress -A                                  # xem host, path, class
kubectl describe ingress <tên>                          # xem backend và sự kiện
kubectl get ingressclass                                # class có tồn tại không
kubectl get pods -n <ns-controller> -l <nhãn-controller>  # controller có đang chạy không
kubectl logs -n <ns-controller> <pod-controller> --tail=100   # log của controller
kubectl get svc <svc-backend> -o yaml                   # cổng của Service backend
```

| Mã | Thường gặp | Hướng xử lý |
|---|---|---|
| 404 | Host hoặc path không khớp, hoặc sai ingress class | Kiểm tra `host`, `pathType`, `ingressClassName` |
| 502 | Controller không kết nối được backend | Kiểm tra Service, targetPort, Pod đã READY chưa |
| 503 | Không có backend khả dụng | Kiểm tra endpoint của Service |
| TLS lỗi | Secret TLS sai hoặc thiếu | Kiểm tra `tls` và tên Secret cùng namespace |

**Lưu ý:** ingress-nginx đã được thông báo ngừng bảo trì (xem mục 17). Với cluster đang dùng, cần kế hoạch di chuyển. [Đã kiểm chứng]

### 42.11. DNS không phân giải

```sh
kubectl -n kube-system get pods -l k8s-app=kube-dns     # CoreDNS có chạy không (nhãn có thể khác)
kubectl -n kube-system logs -l k8s-app=kube-dns --tail=50
kubectl run dns --rm -it --restart=Never --image=busybox:1.36 -- nslookup kubernetes.default
kubectl run dns --rm -it --restart=Never --image=busybox:1.36 -- cat /etc/resolv.conf
```

[Đã kiểm chứng: trang Debug Services có các lệnh nslookup với tên Service, tên có namespace và tên đầy đủ]

**Mẫu tên DNS:**
- Trong cùng namespace: `web`.
- Khác namespace: `web.dev`.
- Đầy đủ: `web.dev.svc.cluster.local`.

**Lỗi thường gặp:** tên ngắn chạy được trong namespace này nhưng không chạy được từ namespace khác. Hoặc ứng dụng dùng sai hậu tố khiến truy vấn tốn thời gian (liên quan đến tùy chọn `ndots` trong `resolv.conf`). [Trí nhớ]

### 42.12. Node ở trạng thái NotReady

```sh
kubectl get nodes                                       # trạng thái Ready/NotReady
kubectl describe node <tên>                             # mục Conditions: MemoryPressure, DiskPressure, PIDPressure, Ready
kubectl get events --field-selector involvedObject.name=<tên>
```

**Trên chính node** (cần quyền truy cập SSH hoặc console):

```sh
systemctl status kubelet                               # kubelet có đang chạy không
journalctl -u kubelet --since "30 min ago" | tail -n 100   # log kubelet gần đây
df -h                                                   # đầy đĩa?
free -h                                                 # thiếu bộ nhớ?
systemctl status containerd                             # runtime (nếu dùng containerd)
```

**Nguyên nhân thường gặp:**
- kubelet dừng hoặc không kết nối được apiserver (chứng chỉ hết hạn, mạng).
- Hết dung lượng đĩa, khiến kubelet đánh dấu `DiskPressure` và đẩy Pod đi.
- Runtime container lỗi.
- Node mất mạng với control plane.

**Khi cần bảo trì node:**

```sh
kubectl cordon <node>                                   # không đặt Pod mới
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data   # sơ tán Pod, tôn trọng PDB
kubectl uncordon <node>                                 # cho phép lại
```

[Đã kiểm chứng: tài liệu Kubernetes có lệnh cordon và drain; chi tiết tham số cần đối chiếu]

### 42.13. Lỗi RBAC (Forbidden)

```sh
kubectl auth can-i list pods -n <ns>                         # tôi có quyền không?
kubectl auth can-i list pods -n <ns> --as <người-dùng>        # kiểm tra quyền của người khác
kubectl get rolebindings,clusterrolebindings -A | grep <tên>  # tìm binding liên quan
kubectl describe role <tên> -n <ns>                          # xem quyền thật sự của Role
```

**Lưu ý:** lỗi `Forbidden` nói rõ tài khoản, động từ, tài nguyên, và namespace. Đọc đủ bốn yếu tố này để biết cần cấp gì. [Suy luận]

### 42.14. Control plane và kết nối kubectl

```sh
kubectl cluster-info                                     # địa chỉ apiserver có phản hồi không
kubectl config current-context                           # đang trỏ đúng cluster không
kubectl get --raw='/readyz?verbose'                      # trạng thái chi tiết của apiserver [Trí nhớ]
kubectl get --raw='/livez?verbose'                       # tình trạng sống của apiserver [Trí nhớ]
```

**Trên cluster tự quản lý**, kiểm tra etcd bằng công cụ `etcdctl`. Ví dụ: `etcdctl endpoint health`, cần cấu hình chứng chỉ đúng. [Trí nhớ]

**Nguyên nhân thường gặp khi kubectl không kết nối:**
- Sai context hoặc kubeconfig cũ.
- Chứng chỉ client hoặc server hết hạn.
- VPN hoặc proxy chặn.
- apiserver quá tải hoặc không khỏe.

### 42.15. Tài nguyên và hiệu năng

```sh
kubectl top pods -A --sort-by=memory                    # Pod dùng nhiều bộ nhớ nhất
kubectl top pods -A --sort-by=cpu                       # Pod dùng nhiều CPU nhất
kubectl top nodes                                       # mức tải node
kubectl get pods -A -o wide --field-selector spec.nodeName=<node>   # Pod trên một node
kubectl describe pod <tên> | grep -E 'OOMKilled|Limits|Requests'
```

**Dấu hiệu:**
- `OOMKilled` hoặc exit code 137: bộ nhớ vượt limit.
- CPU chạm limit liên tục: ứng dụng bị giới hạn (throttle), đánh giá lại `limits.cpu`.
- Node đầy `requests` nhưng CPU thực tế thấp: requests đặt quá cao, lãng phí tài nguyên. [Suy luận]

### 42.16. Bộ lệnh gỡ lỗi nhanh

Chạy theo thứ tự khi không biết bắt đầu từ đâu:

```sh
kubectl get nodes -o wide                               # node có sẵn sàng không
kubectl get pods -A -o wide | grep -v Running | grep -v Completed   # Pod không bình thường
kubectl get events -A --sort-by=.lastTimestamp | tail -n 30       # sự kiện gần nhất
kubectl get pv,pvc -A | grep -v Bound                   # volume chưa gắn
kubectl get svc,endpointslices -A | head -n 50          # dịch vụ và endpoint
kubectl top nodes                                       # tải node
```

### 42.17. Công cụ gỡ lỗi mạng trong cluster

Một Pod tạm có sẵn công cụ mạng giúp kiểm tra kết nối:

```sh
kubectl run netshoot --rm -it --restart=Never --image=nicolaka/netshoot -- bash   # công cụ mạng đầy đủ [Trí nhớ]
# Trong Pod:
curl -v http://web.dev.svc.cluster.local:80/           # kiểm tra HTTP và DNS
dig web.dev.svc.cluster.local                          # kiểm tra DNS
traceroute <ip>                                        # đường đi
```

**Lưu ý:** dùng image gỡ lỗi chỉ trong môi trường được phép. Tên image cần kiểm tra độ tin cậy. [Suy luận]

---

## 43. Runbook sự cố: mẫu ghi chép

Khi có sự cố, ghi lại để điều tra sau:

```
Thời điểm phát hiện:
Triệu chứng (người dùng thấy gì):
Phạm vi (dịch vụ, namespace, node, khu vực):
Thay đổi gần nhất (deploy, cấu hình, nâng cấp):
Lệnh đã chạy và kết quả chính:
Nguyên nhân gốc (đã xác nhận, không phải suy đoán):
Cách khắc phục tạm thời:
Cách khắc phục lâu dài:
Hành động phòng ngừa (alert, test, quy trình):
```

**Nguyên tắc sau sự cố:** tập trung vào hệ thống và quy trình, không quy lỗi cá nhân. [Suy luận]

---

## 44. Phòng ngừa: giảm số sự cố cần gỡ

- **Probes đúng:** readiness cho việc nhận lưu lượng, liveness cho việc khởi động lại, startup cho khởi động chậm.
- **requests và limits hợp lý:** tránh Pod bị đẩy hoặc bị kill vì thiếu tài nguyên.
- **PDB:** tránh bảo trì node làm mất toàn bộ bản sao.
- **Ghim phiên bản image và chart:** tránh lỗi do thay đổi không kiểm soát.
- **Cảnh báo sớm:** cảnh báo khi Pod restart nhiều, khi node NotReady, khi PVC Pending lâu, khi tỷ lệ 5xx tăng.
- **Kiểm tra cấu hình trước khi áp dụng:** `--dry-run=server` và `diff`.
- **Diễn tập:** thử các kịch bản lỗi trên cluster không sản xuất (xóa Pod, cạn tài nguyên, sai cấu hình).

---

# PHẦN K. QUAN SÁT VÀ VẬN HÀNH

## 42. Logging, metrics, tracing

- **Logging:** log container ghi ra stdout và stderr; agent (DaemonSet) gom log về nơi tập trung. [Trí nhớ]
- **Metrics:** Metrics Server cho CPU và bộ nhớ; Prometheus cho số liệu chi tiết. [Trí nhớ]
- **Tracing:** OpenTelemetry (xem tài liệu OpenTelemetry đã viết) để nối trace qua dịch vụ. [Suy luận]

## 43. Sao lưu, nâng cấp, và bảo trì node

- **Sao lưu etcd:** dùng `etcdctl snapshot save` theo tài liệu của cluster. [Trí nhớ]
- **Sao lưu tài nguyên:** công cụ như Velero sao lưu manifest và volume. [Trí nhớ]
- **Nâng cấp:** nâng từng minor version, nâng control plane trước, rồi node. Đọc ghi chú phát hành. [Suy luận]
- **Bảo trì node:** `kubectl cordon`, rồi `kubectl drain` với PDB, rồi bảo trì, rồi `kubectl uncordon`. [Trí nhớ]

---

# PHẦN L. MÔI TRƯỜNG HỌC VÀ THỬ NGHIỆM

## 44. Cluster cục bộ

| Công cụ | Đặc điểm |
|---|---|
| **kind** | Cluster Kubernetes chạy trong container Docker, nhẹ, phù hợp CI [Trí nhớ] |
| **minikube** | Cluster cục bộ trong VM hoặc container, có nhiều add-on [Trí nhớ] |
| **k3d / k3s** | Bản Kubernetes nhẹ [Trí nhớ] |

```sh
kind create cluster --name hoc                  # tạo cluster tên hoc
kubectl cluster-info --context kind-hoc         # kiểm tra kết nối
kind delete cluster --name hoc                  # xóa khi không dùng nữa
```

[Kiểm tra cú pháp và yêu cầu Docker theo tài liệu của kind và minikube]

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Cluster | Tập hợp control plane và node |
| Control plane | Các thành phần quyết định trạng thái cluster |
| Node | Máy (ảo hoặc vật lý) chạy Pod |
| Pod | Đơn vị nhỏ nhất được lập lịch, chứa một hoặc nhiều container |
| Label / Selector | Nhãn và biểu thức chọn đối tượng |
| Deployment | Quản lý ReplicaSet và cập nhật theo đợt |
| ReplicaSet | Đảm bảo số lượng Pod |
| StatefulSet | Ứng dụng có trạng thái, Pod có tên ổn định |
| DaemonSet | Một Pod trên mỗi node |
| Job / CronJob | Tác vụ một lần / định kỳ |
| Service | Địa chỉ ổn định cho tập Pod |
| Ingress / Gateway | Định tuyến HTTP từ bên ngoài |
| NetworkPolicy | Giới hạn lưu lượng giữa Pod |
| ConfigMap / Secret | Cấu hình / bí mật |
| PV / PVC / StorageClass | Lưu trữ bền vững |
| Requests / Limits | Tài nguyên tối thiểu / tối đa |
| Probe | Kiểm tra sức khỏe container |
| HPA / VPA / PDB | Tự co giãn ngang / dọc, và giới hạn gián đoạn |
| RBAC | Phân quyền dựa trên vai trò |
| ServiceAccount | Danh tính của tiến trình trong Pod |
| Taint / Toleration | Đẩy Pod khỏi node / cho phép Pod đến node |
| Reconciliation | Vòng lặp đưa trạng thái thực tế về trạng thái mong muốn |
| CNI / CRI / CSI | Giao diện mạng / runtime / lưu trữ |
| Helm / Kustomize | Quản lý gói / tùy biến manifest |
| GitOps | Git là nguồn sự thật cho cluster |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **Kubernetes Releases**: https://kubernetes.io/releases/ [đã đọc. Ghi nhận: 1.37 là mới nhất được hỗ trợ (bản vá 1.37.1, 2026-09-15); 1.36, 1.35, 1.34 được hỗ trợ; chính sách khoảng một năm cho các bản từ 1.19]
- **Kubernetes Architecture (Components)**: https://kubernetes.io/docs/concepts/architecture/ [đã đọc một phần. Ghi nhận: vai trò kube-apiserver, etcd, kube-scheduler, kube-controller-manager, cloud-controller-manager, kubelet, kube-proxy]
- **Thông báo ngừng bảo trì ingress-nginx** (blog Kubernetes, 2025-11-11): https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/ [đã đọc một phần. Ghi nhận: bảo trì nỗ lực đến tháng 3 năm 2026, sau đó không có bản phát hành, bản vá, hay cập nhật bảo mật. Phần khuyến nghị thay thế không được đọc đầy đủ]

## N.2. Nguồn chưa đối chiếu

- Tài liệu chi tiết về Pod, Deployment, Service, Ingress, Gateway API, NetworkPolicy, RBAC, Pod Security Standards.
- Tài liệu kubectl (cheat sheet và các lệnh cụ thể), đặc biệt `kubectl auth whoami` và các cờ mới.
- Tài liệu Helm, Kustomize, Argo CD, Flux, Velero.
- Tài liệu kind, minikube, k3d.
- Phần còn lại của trang kiến trúc (kubelet, container runtime, kube-proxy chi tiết).

## N.3. Điểm cần xác minh

1. **Thay thế cho ingress-nginx:** bài công bố đã đọc không nêu rõ khuyến nghị. Các tiêu đề liên quan gợi ý Gateway API, nhưng cần đọc bài đầy đủ hoặc tài liệu chính thức.
2. **`apiVersion` và trường cấu hình:** các manifest là minh họa theo kiến thức nền. Cần kiểm tra bằng `kubectl explain` trên cluster phiên bản 1.37.
3. **Các cờ kubectl** như `kubectl auth whoami` và `--field-selector`: cần xác nhận trên phiên bản đang dùng.
4. **Chuẩn Pod Security và tên trường securityContext:** kiến thức nền, cần đối chiếu tài liệu.
5. **Tên repo và chart Helm** trong ví dụ là minh họa.
6. **Ví dụ GitHub Actions và kubectl** trong pipeline cần thay bằng cơ chế xác thực an toàn thực tế.
7. **Các manifest chưa được áp dụng thử.** Nên kiểm tra trên kind trước khi dùng.

---

**Tóm tắt để tra nhanh:**
- **Mô hình:** khai báo trạng thái mong muốn, vòng lặp điều hòa đưa thực tế về mong muốn.
- **Control plane:** apiserver (cửa vào), etcd (dữ liệu), scheduler (chọn node), controller-manager (điều hòa), cloud-controller-manager (tích hợp cloud).
- **Node:** kubelet (chạy Pod), kube-proxy (mạng Service), container runtime (chạy container).
- **Đối tượng cốt lõi:** Pod, Deployment, Service, ConfigMap, Secret, PVC, Ingress/Gateway, NetworkPolicy, RBAC.
- **Workflow:** build image có tag, kiểm tra bằng `--dry-run=server` và `diff`, `apply`, `rollout status`, `rollout undo` khi lỗi.
- **Gỡ lỗi:** `get` → `describe` (Events) → `logs --previous` → `get events`.
- **Cẩn trọng:** không dùng tag `latest`, đặt requests và limits, không lưu kubeconfig có quyền quản trị trong mã nguồn, và kiểm tra trạng thái ingress-nginx.
