# Tài liệu Helm: trình quản lý gói cho Kubernetes

Tài liệu gồm: khái niệm và kiến trúc, cấu trúc chart, template và values, vòng đời release, quy trình làm việc, câu lệnh, kiểm thử, xử lý sự cố và nguồn tham khảo.

Ghi chú về nguồn và mức độ xác minh:
- **[Đã đối chiếu]:** phiên bản mới nhất trên trang release của GitHub; cấu trúc thư mục chart và trường bắt buộc trong Chart.yaml trên trang tài liệu chart của helm.sh; các cờ của `helm upgrade` trên trang tham chiếu lệnh của helm.sh; các thay đổi chính của Helm 4 trên trang tổng quan của helm.sh.
- **[Chưa đối chiếu trong lần viết này]:** cơ chế lưu trữ release, thứ tự ưu tiên của values, hook, và các lệnh còn lại. Viết từ kiến thức nền, cần kiểm tra lại với phiên bản đang dùng.
- Các ví dụ YAML và lệnh chưa được chạy thử trên cụm thật.

Mục lục:
- [Phần 1. Helm là gì và giải quyết vấn đề gì](#phần-1-helm-là-gì-và-giải-quyết-vấn-đề-gì)
- [Phần 2. Phiên bản: Helm 3 và Helm 4](#phần-2-phiên-bản-helm-3-và-helm-4)
- [Phần 3. Khái niệm cốt lõi và kiến trúc](#phần-3-khái-niệm-cốt-lõi-và-kiến-trúc)
- [Phần 4. Cấu trúc một chart](#phần-4-cấu-trúc-một-chart)
- [Phần 5. Template, values và thứ tự ưu tiên](#phần-5-template-values-và-thứ-tự-ưu-tiên)
- [Phần 6. Vòng đời release và quy trình làm việc](#phần-6-vòng-đời-release-và-quy-trình-làm-việc)
- [Phần 7. Câu lệnh](#phần-7-câu-lệnh)
- [Phần 8. Dependency, repository và OCI](#phần-8-dependency-repository-và-oci)
- [Phần 9. Hook và kiểm thử](#phần-9-hook-và-kiểm-thử)
- [Phần 10. Xử lý sự cố](#phần-10-xử-lý-sự-cố)
- [Phần 11. Bảo mật và thực hành tốt](#phần-11-bảo-mật-và-thực-hành-tốt)
- [Phần 12. Thuật ngữ](#phần-12-thuật-ngữ)
- [Phần 13. Nguồn và điểm cần xác minh](#phần-13-nguồn-và-điểm-cần-xác-minh)

---

## Phần 1. Helm là gì và giải quyết vấn đề gì

Một ứng dụng trên Kubernetes thường gồm nhiều manifest: Deployment, Service, ConfigMap, Secret, Ingress. Khi triển khai ở nhiều môi trường (dev, staging, production), mỗi môi trường cần giá trị khác nhau (số bản sao, tên miền, tài nguyên). Nếu copy-paste YAML, dễ sai sót và khó cập nhật đồng bộ.

Helm giải quyết bằng cách:
- **Đóng gói** các manifest thành một đơn vị gọi là **chart**.
- **Tham số hóa** bằng Go template và file **values** để một chart dùng được cho nhiều môi trường.
- **Quản lý vòng đời** ứng dụng đã cài: cài, nâng cấp, quay lại phiên bản cũ, gỡ bỏ.
- **Chia sẻ** chart qua repository hoặc OCI registry.

Helm thường được mô tả là "trình quản lý gói cho Kubernetes", tương tự apt hay npm nhưng cho ứng dụng trên cụm.

## Phần 2. Phiên bản: Helm 3 và Helm 4

[Đã đối chiếu] Trên trang release của GitHub, các bản mới nhất tính đến thời điểm viết gồm Helm v4.3.1 (đánh dấu Latest, phát hành ngày 15/10/2026), Helm v3.22.1 (cùng ngày) và Helm v4.3.0 (ngày 09/09/2026).

[Đã đối chiếu] Trang tổng quan của helm.sh mô tả Helm 4 là bản nâng cấp lớn, có thay đổi phá vỡ tương thích, và liệt kê:

| Thay đổi | Ý nghĩa khi chuyển đổi |
|---|---|
| Post-renderer yêu cầu tên plugin | Cấu hình post-renderer cũ có thể cần sửa |
| `helm registry login` chỉ nhận tên miền | Không dùng được địa chỉ kiểu IP:port như trước |
| Đổi tên cờ, ví dụ `--atomic` thành `--rollback-on-failure` | Script cũ cần cập nhật |
| Plugin WebAssembly | Hướng phát triển plugin mới |
| Theo dõi tài nguyên bằng kstatus | Cách chờ tài nguyên sẵn sàng có thể khác |
| Cài đặt dùng server-side apply mặc định | Có thể gặp xung đột field khi chuyển từ client-side apply |
| Chart v3 (thử nghiệm) | Chart v2 hiện tại vẫn hoạt động |

Khuyến nghị: với dự án mới, kiểm tra xem có cần Helm 4 hay không. Với hệ thống đang chạy Helm 3, nên đọc kỹ ghi chú chuyển đổi trước khi nâng cấp.

## Phần 3. Khái niệm cốt lõi và kiến trúc

### 3.1. Ba khái niệm chính

| Khái niệm | Vai trò | Ví dụ |
|---|---|---|
| Chart | Gói mẫu manifest kèm metadata và values mặc định | Chart `nginx` gồm Deployment, Service, values |
| Release | Một bản cài đặt cụ thể của chart trong một namespace | Release `web-prod` cài từ chart `nginx` |
| Repository | Nơi lưu và chia sẻ chart | Repository HTTP hoặc OCI registry |

Một chart có thể được cài nhiều lần, mỗi lần thành một release khác nhau với tên và values khác nhau.

### 3.2. Kiến trúc

- **Helm CLI:** công cụ dòng lệnh chạy trên máy người dùng hoặc trong CI. Đọc chart, render template, gọi API của Kubernetes.
- **Cụm Kubernetes:** nơi các manifest được áp dụng. Helm không cần chạy một dịch vụ (server-side component) riêng như Tiller trong Helm 2.
- **Lưu trữ release:** Helm lưu thông tin release (manifest đã render, values, lịch sử revision) vào cụm Kubernetes. Trong Helm 3 mặc định dùng Secret; cơ chế cụ thể trong Helm 4 cần kiểm tra lại (chưa đối chiếu trong lần viết này).

Lưu ý: Tiller của Helm 2 đã bị loại bỏ từ Helm 3. Đây là thay đổi lịch sử quan trọng khi đọc tài liệu cũ.

## Phần 4. Cấu trúc một chart

[Đã đối chiếu] Theo tài liệu chart của helm.sh:

```
mychart/
├── Chart.yaml          # bắt buộc: metadata
├── values.yaml         # giá trị mặc định
├── values.schema.json  # (tùy chọn) JSON Schema để kiểm tra values
├── charts/             # (tùy chọn) chart phụ thuộc, do Helm quản lý
├── crds/               # (tùy chọn) CustomResourceDefinition, YAML thuần, không template
├── templates/          # Go template render thành manifest
│   ├── NOTES.txt       # (tùy chọn) hướng dẫn in ra sau khi cài
│   ├── _helpers.tpl    # hàm đặt tên dùng chung (theo quy ước)
│   └── deployment.yaml
├── README.md           # (tùy chọn) tài liệu hiển thị trên các hub
└── LICENSE             # (tùy chọn) giấy phép
```

### 4.1. Chart.yaml

Các trường bắt buộc: `apiVersion`, `name`, `version`. Version nên theo SemVer 2. `helm package` yêu cầu tên file gói khớp với version trong Chart.yaml.

Ví dụ:

```yaml
apiVersion: v2          # v2: chart cần Helm 3 trở lên; dependency khai báo trong chính file này
name: webapp            # tên chart
description: Ứng dụng web mẫu
type: application       # application hoặc library
version: 0.1.0          # version của chart (SemVer)
appVersion: "1.16.0"    # version của ứng dụng bên trong, chỉ để hiển thị
```

Giải thích từng dòng:
- `apiVersion: v2`: chart v2 yêu cầu Helm 3 trở lên. Chart v1 vẫn cài được bằng Helm 3, nhưng dependency được khai báo ở file requirements.yaml riêng.
- `name`: tên chart, cũng thường là tên thư mục.
- `type`: `application` cài được; `library` chỉ cung cấp helper, không tạo manifest.
- `version`: thay đổi mỗi khi chart thay đổi, dùng để chọn chart khi cài.
- `appVersion`: không ảnh hưởng đến cách Helm xử lý; dùng để ghi nhãn phiên bản ứng dụng.

### 4.2. values.yaml

File chứa giá trị mặc định. Người dùng ghi đè khi cài bằng `-f` hoặc `--set`.

```yaml
replicaCount: 2         # số bản sao mặc định
image:
  repository: nginx     # tên image
  tag: "1.27"           # tag cố định; tránh dùng latest
service:
  type: ClusterIP       # loại Service
  port: 80
```

## Phần 5. Template, values và thứ tự ưu tiên

### 5.1. Template

File trong `templates/` là Go template, được render với values và các đối tượng dựng sẵn như `.Release`, `.Chart`, `.Values`. Helm mở rộng với hàm từ thư viện Sprig.

Ví dụ `templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web          # tên có tiền tố tên release
spec:
  replicas: {{ .Values.replicaCount }}   # lấy từ values
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

Giải thích:
- `{{ .Release.Name }}`: tên release do người dùng đặt khi cài. Dùng để tránh trùng tên khi cài nhiều lần.
- `{{ .Values.replicaCount }}`: đọc giá trị từ values đã ghép.
- `"{{ ... }}:{{ ... }}"`: dấu ngoặc kép giúp YAML coi kết quả là chuỗi.

### 5.2. Hàm và điều kiện thường dùng

```yaml
{{- if .Values.ingress.enabled }}     # chỉ render khi bật
...
{{- end }}
{{ include "webapp.fullname" . | nindent 4 }}   # gọi helper từ _helpers.tpl
{{ .Values.image.tag | default "1.27" }}       # giá trị mặc định nếu trống
```

- `{{-` ở đầu: bỏ khoảng trắng phía trước, tránh dòng trống thừa trong YAML.
- `include`: gọi hàm đã định nghĩa trong `_helpers.tpl`. Cần dùng `include` thay vì `template` để có thể pipe kết quả sang hàm khác.
- `nindent 4`: thụt đầu dòng 4 khoảng trắng và thêm dòng mới phía trước.

### 5.3. Thứ tự ưu tiên của values

Giá trị được ghép theo thứ tự, phía sau ghi đè phía trước (từ thấp đến cao):

1. `values.yaml` trong chart (thấp nhất)
2. Các file `-f` theo thứ tự xuất hiện, file bên phải thắng
3. Các tham số `--set`, `--set-string`, `--set-file` (cao nhất)

Ghi chú: trong tài liệu tham chiếu lệnh upgrade, cờ `--set` cho phép lặp lại và giá trị cuối cùng thắng; `-f` có thể lặp và file bên phải thắng. Các cơ chế `--reuse-values` và `--reset-values` ảnh hưởng đến việc giữ hay bỏ values của release cũ (xem Phần 7).

## Phần 6. Vòng đời release và quy trình làm việc

### 6.1. Vòng đời

```
Chart (thư mục hoặc gói .tgz)
  -> helm template (render, không gửi lên cụm)
  -> helm install (tạo release revision 1)
  -> helm upgrade (tạo revision 2, 3, ...)
  -> helm rollback (tạo revision mới trỏ về nội dung cũ)
  -> helm uninstall (gỡ tài nguyên, lịch sử tùy cờ giữ lại)
```

### 6.2. Quy trình làm việc đề xuất

1. **Tạo chart:** `helm create webapp` tạo khung mẫu.
2. **Chỉnh sửa templates và values.**
3. **Kiểm tra cú pháp:** `helm lint webapp`.
4. **Xem kết quả render:** `helm template webapp ./webapp -f values-prod.yaml`.
5. **Mô phỏng phía cụm:** `helm install --dry-run`.
6. **Cài đặt hoặc nâng cấp:** `helm upgrade --install web-prod ./webapp -n prod -f values-prod.yaml`.
7. **Kiểm tra:** `helm status`, `kubectl get pods`, `helm test` (nếu có test).
8. **Quay lại khi lỗi:** `helm rollback web-prod <revision>`.

Lưu ý: nên đặt `--history-max` để giới hạn số revision được lưu, tránh lịch sử phình to. Mặc định trong tài liệu là 10.

## Phần 7. Câu lệnh

### 7.1. Nhóm chính

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `helm create <tên>` | Tạo chart mẫu | `helm create webapp` |
| `helm lint <chart>` | Kiểm tra lỗi cấu trúc và template | `helm lint ./webapp` |
| `helm template <tên> <chart>` | Render manifest ra stdout, không cài | `helm template web ./webapp -f values-prod.yaml` |
| `helm install <tên> <chart>` | Cài release mới | `helm install web ./webapp -n dev --create-namespace` |
| `helm upgrade <tên> <chart>` | Nâng cấp release | `helm upgrade web ./webapp --set replicaCount=3` |
| `helm upgrade --install` | Cài nếu chưa có, nâng cấp nếu đã có | `helm upgrade --install web ./webapp -n prod` |
| `helm rollback <tên> <revision>` | Quay về revision trước | `helm rollback web 2` |
| `helm uninstall <tên>` | Gỡ release | `helm uninstall web -n dev` |
| `helm list` | Liệt kê release | `helm list -A` |
| `helm status <tên>` | Trạng thái release | `helm status web -n prod` |
| `helm history <tên>` | Lịch sử revision | `helm history web` |
| `helm get values <tên>` | Xem values đang dùng | `helm get values web --all` |
| `helm get manifest <tên>` | Xem manifest đã áp dụng | `helm get manifest web` |

### 7.2. Các cờ quan trọng của `helm upgrade` [Đã đối chiếu]

| Cờ | Tác dụng |
|---|---|
| `-i`, `--install` | Cài nếu release chưa tồn tại |
| `--rollback-on-failure` | Quay về bản thành công gần nhất nếu nâng cấp lỗi (thay cho `--atomic` ở Helm 3) |
| `--wait` | Chờ tài nguyên sẵn sàng. Giá trị chấp nhận: `watcher`, `hookOnly`, `legacy`. Mặc định `hookOnly` |
| `--timeout` | Thời gian tối đa cho mỗi thao tác. Mặc định 5 phút |
| `--set` | Đặt giá trị trên dòng lệnh. Có thể lặp lại |
| `-f`, `--values` | Nạp file values hoặc URL. Có thể lặp lại |
| `--version` | Chọn version chart hoặc khoảng version |
| `--reuse-values` | Giữ values của release trước, gộp với override mới |
| `--reset-values` | Dùng giá trị mặc định của chart, bỏ values cũ |
| `--reset-then-reuse-values` | Dùng mặc định của chart, rồi áp values cũ và override mới |
| `--dry-run` | Mô phỏng. Giá trị: `none` (mặc định), `client`, `server`. Có thể lộ Secret, dùng `--hide-secret` để ẩn |
| `--history-max` | Số revision lưu tối đa. Mặc định 10; 0 là không giới hạn |
| `--create-namespace` | Tạo namespace nếu chưa có (cùng với install) |

Lưu ý: tài liệu tham chiếu không liệt kê `--atomic` và `--force`. Cờ `--force-replace` và `--force-conflicts` là các cờ liên quan đến cách thay thế tài nguyên và xung đột server-side apply; cần đọc kỹ trước khi dùng.

### 7.3. Lệnh mẫu có chú thích

```bash
# Cài hoặc nâng cấp release "web" trong namespace "prod"
# --install: tạo mới nếu chưa có
# --wait: chờ pod sẵn sàng
# --timeout 5m: tối đa 5 phút
# --rollback-on-failure: tự quay lại nếu lỗi
# -f values-prod.yaml: ghi đè values cho môi trường production
# --set image.tag=1.27.2: ghi đè một giá trị, ưu tiên cao nhất
helm upgrade --install web ./webapp \
  -n prod --create-namespace \
  --wait --timeout 5m \
  --rollback-on-failure \
  -f values-prod.yaml \
  --set image.tag=1.27.2
```

Ghi chú: lệnh trên chưa được chạy thử. Trước khi dùng trên production, chạy `helm template` hoặc `--dry-run` để xem kết quả.

## Phần 8. Dependency, repository và OCI

### 8.1. Chart phụ thuộc

Chart có thể phụ thuộc vào chart khác (ví dụ Redis, PostgreSQL). Với chart v2, khai báo trong `Chart.yaml`:

```yaml
dependencies:
  - name: redis                  # tên chart phụ thuộc
    version: "~19.0.0"           # khoảng version
    repository: https://charts.example.com   # nơi lấy chart (ví dụ)
    condition: redis.enabled     # chỉ cài khi giá trị này là true
```

Lệnh:
- `helm dependency update ./webapp`: tải chart phụ thuộc vào thư mục `charts/`.
- `helm dependency list ./webapp`: xem danh sách và trạng thái.

Cờ `condition` giúp bật hoặc tắt chart phụ thuộc qua values.

### 8.2. Repository (HTTP)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami   # thêm repository (ví dụ)
helm repo update                                            # cập nhật danh sách chart
helm search repo nginx                                      # tìm chart
helm show values bitnami/nginx                              # xem values mặc định
```

Lưu ý: địa chỉ repository trong ví dụ chỉ là minh họa. Trước khi dùng, kiểm tra địa chỉ chính thức của nhà phát hành chart.

### 8.3. OCI registry

Helm hỗ trợ đóng gói và lấy chart từ OCI registry. [Đã đối chiếu] Trang release của Helm 4 ghi có cài đặt từ digest của OCI chart. Một lệnh minh họa:

```bash
helm pull oci://registry.example.com/charts/webapp --version 0.1.0   # tải chart từ OCI
helm install web oci://registry.example.com/charts/webapp --version 0.1.0
```

Đăng nhập registry: `helm registry login`. Theo ghi chú Helm 4, lệnh này chỉ nhận tên miền (không nhận IP:port).

## Phần 9. Hook và kiểm thử

### 9.1. Hook

Hook là manifest có annotation `helm.sh/hook`, chạy vào các thời điểm nhất định của vòng đời, ví dụ `pre-install`, `post-install`, `pre-upgrade`, `post-delete`. Thường dùng cho Job chạy migration cơ sở dữ liệu.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-migrate
  annotations:
    "helm.sh/hook": pre-upgrade                # chạy trước khi nâng cấp
    "helm.sh/hook-weight": "0"                 # thứ tự khi có nhiều hook
    "helm.sh/hook-delete-policy": hook-succeeded  # xóa Job khi thành công
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["./migrate"]
```

Giải thích: hook chạy trước khi tài nguyên chính được nâng cấp. Nếu Job thất bại, nâng cấp dừng lại và có thể kích hoạt rollback tùy cờ.

### 9.2. Kiểm thử

`helm test <tên>` chạy các Pod có annotation `helm.sh/hook: test`. Thường là Pod gọi thử Service để kiểm tra kết nối.

Lưu ý: các chi tiết về hook policy và test cần đối chiếu lại với tài liệu của phiên bản đang dùng.

## Phần 10. Xử lý sự cố

| Triệu chứng | Nguyên nhân thường gặp | Cách kiểm tra | Hướng xử lý |
|---|---|---|---|
| `helm lint` báo lỗi cú pháp | Thiếu dấu ngoặc, thụt lề sai trong template | Chạy `helm lint --strict` và xem dòng báo lỗi | Sửa template; kiểm tra `{{- }}` và khoảng trắng |
| Render ra manifest sai | Values không đúng kiểu, thiếu key | `helm template ... --debug` | Kiểm tra values; dùng `default`; thêm `values.schema.json` |
| `Error: UPGRADE FAILED: another operation is in progress` | Lần cài trước bị treo hoặc dừng giữa chừng | `helm status`, `helm history` | Kiểm tra revision pending; chạy rollback về revision thành công gần nhất |
| `Error: UPGRADE FAILED: ... immutable field` | Thay đổi trường không cho phép sửa (ví dụ selector của Deployment) | Đọc thông báo lỗi và manifest cũ | Đổi tên release hoặc xóa tài nguyên cũ có kiểm soát; tránh đổi selector |
| `Error: rendered manifests contain a resource that already exists` | Tài nguyên đã tồn tại không do Helm quản lý | `kubectl get <kind> <tên> -o yaml` xem annotation | Thêm annotation và label của Helm, hoặc đổi tên tài nguyên |
| `Error: Kubernetes cluster unreachable` | Kubeconfig sai, context sai | `kubectl cluster-info`, `helm env` | Chọn đúng context; kiểm tra KUBECONFIG |
| Release kẹt ở `pending-install` hoặc `pending-upgrade` | Tiến trình trước bị ngắt | `helm history <tên>` | Rollback hoặc uninstall rồi cài lại (cân nhắc dữ liệu) |
| Pod không sẵn sàng sau `--wait` | Lỗi ứng dụng, probe sai, thiếu config | `kubectl describe pod`, `kubectl logs` | Sửa ứng dụng hoặc values; xem thêm tài liệu Kubernetes |
| `helm dependency update` lỗi | Repository không truy cập được, version không khớp | Thử `helm repo update`, kiểm tra tên và version | Sửa URL, khoảng version, hoặc chứng chỉ TLS |
| Lệnh `helm` cũ không hoạt động với Helm 4 | Đổi tên cờ, post-renderer, registry login | Đọc ghi chú chuyển đổi | Cập nhật script theo Phần 2 |
| Values không có hiệu lực | Ghi đè bởi `--set` hoặc file bên phải; `--reuse-values` giữ giá trị cũ | `helm get values <tên> --all` | Kiểm tra thứ tự ưu tiên (Phần 5.3) |

Nguyên tắc gỡ lỗi:
1. Render trước: `helm template` để xem manifest cuối cùng.
2. So sánh: `helm get manifest` (đã áp dụng) với kết quả render.
3. Xem trạng thái: `helm status`, `helm history`.
4. Xem tài nguyên: `kubectl describe` và `kubectl logs` như khi debug Kubernetes thông thường.

## Phần 11. Bảo mật và thực hành tốt

- **Không lưu Secret dạng thô trong values.yaml.** Dùng công cụ quản lý secret (ví dụ Sealed Secrets, External Secrets) hoặc tham chiếu Secret đã có sẵn.
- **Cẩn thận với `--dry-run=server` và `helm get values`:** có thể lộ giá trị nhạy cảm. Dùng `--hide-secret` khi có.
- **Ghim version chart và image tag.** Tránh `latest`.
- **Dùng `--rollback-on-failure` và `--wait`** cho môi trường quan trọng.
- **Kiểm tra chart từ nguồn không tin cậy** trước khi cài: đọc template và values, không chỉ chạy.
- **Tách values theo môi trường:** `values-dev.yaml`, `values-prod.yaml`, lưu trong kho mã và được review.
- **Giới hạn quyền RBAC** của người và CI được phép cài release.

## Phần 12. Thuật ngữ

| Thuật ngữ | Giải thích |
|---|---|
| Chart | Gói gồm template, values và metadata |
| Release | Một lần cài chart trong một namespace |
| Revision | Số thứ tự phiên bản của một release |
| Values | Tham số cấu hình để render template |
| Template | File Go template sinh manifest |
| Helper | Hàm đặt tên hoặc tái sử dụng định nghĩa, thường trong `_helpers.tpl` |
| Hook | Manifest chạy vào thời điểm nhất định của vòng đời |
| Repository | Nơi lưu chart (HTTP hoặc OCI) |
| Dependency | Chart phụ thuộc, nằm trong `charts/` |
| Post-renderer | Bước xử lý manifest sau khi render (cơ chế plugin) |
| SemVer | Quy ước số phiên bản major.minor.patch |

## Phần 13. Nguồn và điểm cần xác minh

### 13.1. Đã đối chiếu
- Trang release của Helm trên GitHub (phiên bản mới nhất).
- Trang tài liệu chart của helm.sh (cấu trúc thư mục, Chart.yaml, apiVersion v1 và v2).
- Trang tham chiếu lệnh `helm upgrade` của helm.sh (các cờ).
- Trang tổng quan của helm.sh (thay đổi của Helm 4).

### 13.2. Chưa đối chiếu trực tiếp
- Cơ chế lưu trữ release trong Helm 4 và chi tiết lưu Secret.
- Thứ tự và chi tiết hook, hook-delete-policy, helm test.
- Cú pháp `_helpers.tpl`, hàm Sprig cụ thể, `values.schema.json`.
- Lệnh `helm rollback`, `helm dependency`, `helm registry login` chi tiết.
- Các ví dụ YAML và bash trong tài liệu: chưa chạy thử trên cụm thật.
- Nội dung thông báo lỗi trong bảng Phần 10 viết từ kiến thức nền; chuỗi chính xác có thể khác theo phiên bản.

### 13.3. Nguồn nên đọc thêm
- Tài liệu chính thức: helm.sh/docs
- Ghi chú chuyển đổi Helm 3 sang Helm 4 trong tài liệu chính thức
- Kho mã nguồn chính thức trên GitHub (helm/helm)

Hết tài liệu Helm.
