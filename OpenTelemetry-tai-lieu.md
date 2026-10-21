# OpenTelemetry: tài liệu tổng quan, kiến trúc, và thực hành

> **Viết cho:** kỹ sư phần mềm, SRE, và người vận hành hệ thống muốn hiểu OpenTelemetry (OTel) từ khái niệm đến cấu hình thực tế.
> **Cách đọc:** Phần A và B là khái niệm nền. Phần C đến E là kiến trúc và thực hành. Mỗi đoạn mã có chú thích từng dòng ngay bên cạnh.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với tài liệu chính thức; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các ví dụ mã chưa được chạy thử. Tên gói, hàm, và cấu hình có thể thay đổi theo phiên bản SDK. Kiểm tra tài liệu của từng ngôn ngữ trước khi dùng.

---

## Mục lục

- **Phần A. OpenTelemetry là gì và vì sao cần**
  - 1. Observability và telemetry
  - 2. Vấn đề OpenTelemetry giải quyết
  - 3. Trạng thái các tín hiệu và độ ổn định
- **Phần B. Các khái niệm cốt lõi**
  - 4. Trace, span, và context
  - 5. Metric
  - 6. Log
  - 7. Resource và attribute
  - 8. Baggage
  - 9. Semantic conventions
  - 10. Propagation: truyền context qua dịch vụ
- **Phần C. Kiến trúc**
  - 11. API, SDK, và instrumentation
  - 12. OTLP: giao thức truyền dữ liệu
  - 13. Collector: receiver, processor, exporter, pipeline
  - 14. Triển khai: agent và gateway
- **Phần D. Thực hành: instrumentation**
  - 15. Tự động (auto) và thủ công (manual) instrumentation
  - 16. Tạo span trong Python
  - 17. Metric trong Python
  - 18. Log và liên kết với trace
  - 19. Sampling
- **Phần E. Thực hành: Collector và cấu hình**
  - 20. Cấu hình Collector
  - 21. Chạy Collector bằng Docker
  - 22. Kiểm tra và gỡ lỗi
- **Phần F. Kiểm thử và tự động hóa**
  - 23. Kiểm thử instrumentation
  - 24. Kiểm tra cấu hình trong CI
- **Phần G. Vận hành và an toàn**
  - 25. Chi phí, cardinality, và dữ liệu nhạy cảm
  - 26. Triển khai trên Kubernetes
- **Phần H. Vai trò của OpenTelemetry trong hệ sinh thái**
  - 27. So sánh với các giải pháp khác
  - 28. Các điểm cần cẩn trọng
- **Phần I. Thuật ngữ**
- **Phần J. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. OPENTELEMETRY LÀ GÌ VÀ VÌ SAO CẦN

## 1. Observability và telemetry

**Telemetry** là dữ liệu đo từ hệ thống đang chạy: thời gian xử lý, số lỗi, nhật ký sự kiện, đường đi của một yêu cầu.

**Observability (khả năng quan sát)** là mức độ bạn có thể hiểu trạng thái bên trong hệ thống từ dữ liệu bên ngoài, mà không cần sửa mã để đoán lỗi. [Trí nhớ]

Ba loại tín hiệu cổ điển thường được gọi là "ba trụ cột của observability":
- **Log:** sự kiện rời rạc có dấu thời gian.
- **Metric:** số đo tổng hợp theo thời gian.
- **Trace:** đường đi của một yêu cầu qua nhiều dịch vụ.

## 2. Vấn đề OpenTelemetry giải quyết

Trước OpenTelemetry, mỗi nhà cung cấp giám sát có thư viện và định dạng riêng. Hệ quả:
- Đổi nhà cung cấp phải sửa lại mã ứng dụng.
- Mỗi ngôn ngữ và framework có cách đo khác nhau.
- Khó liên kết dữ liệu từ nhiều nguồn.

**OpenTelemetry** là dự án mã nguồn mở thuộc CNCF, cung cấp bộ API, SDK, giao thức, và công cụ **chuẩn hóa** cách thu thập và gửi telemetry. Ứng dụng viết instrumentation một lần, rồi gửi dữ liệu đến bất kỳ backend nào hỗ trợ OTLP. [Trí nhớ về nguồn gốc CNCF; Đã kiểm chứng về mục tiêu qua các trang chính thức]

## 3. Trạng thái các tín hiệu và độ ổn định

Trang khái niệm chính thức nhóm các tín hiệu như sau: [Đã kiểm chứng]

| Tín hiệu | Trạng thái theo tài liệu | Vai trò |
|---|---|---|
| **Traces** | Đang được hỗ trợ | Đường đi của yêu cầu qua ứng dụng |
| **Metrics** | Đang được hỗ trợ | Số đo lúc chạy |
| **Logs** | Đang được hỗ trợ | Sự kiện riêng lẻ |
| **Baggage** | Đang được hỗ trợ | Dữ liệu ngữ cảnh truyền giữa các tín hiệu và dịch vụ |
| **Profiles** | Đang phát triển | Mức sử dụng tài nguyên theo mã |
| **Events** | Đang phát triển | Một dạng log đặc biệt |

**Vòng đời độ ổn định** theo tài liệu đặc tả: [Đã kiểm chứng]

| Mức | Ý nghĩa | Ràng buộc |
|---|---|---|
| **Development** | Giai đoạn đầu, có thể thay đổi lớn | API và SDK có thể thay đổi không tương thích |
| **Stable** | Sẵn sàng dùng lâu dài | Không thay đổi không tương thích trong cùng phiên bản lớn |
| **Deprecated** | Đang bị thay thế | Vẫn được hỗ trợ theo cam kết như Stable, đến khi bị gỡ |
| **Removed** | Đã gỡ bỏ | Gỡ bỏ cần tăng phiên bản lớn |

**Lưu ý:** với mỗi ngôn ngữ, API, SDK, và semantic conventions có thể ở các mức khác nhau. Kiểm tra trạng thái của từng thành phần bạn dùng. [Đã kiểm chứng]

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Trace, span, và context

- **Trace:** toàn bộ đường đi của một yêu cầu, gồm nhiều span.
- **Span:** một đơn vị công việc có tên, thời gian bắt đầu và kết thúc, thuộc tính, và quan hệ cha con với span khác.
- **Trace ID** và **Span ID:** định danh duy nhất để nối các span lại với nhau.
- **Context:** thông tin mang theo (trace ID, span ID hiện tại) để span con biết mình thuộc đâu.

```
Trace: POST /dat-hang  (trace_id = abc123)
├── Span: xac-thuc-nguoi-dung         (span_id = 001)
├── Span: kiem-tra-ton-kho            (span_id = 002)
│   └── Span: truy-van-co-so-du-lieu  (span_id = 003)
└── Span: tao-don-hang                (span_id = 004)
```

Mỗi span có: [Trí nhớ]
- **Tên:** mô tả thao tác, ví dụ `GET /api/san-pham`.
- **Thời gian:** bắt đầu, kết thúc, từ đó tính độ trễ.
- **Trạng thái:** `OK`, `ERROR`, hoặc `UNSET`.
- **Sự kiện (events):** các mốc thời gian trong span.
- **Liên kết (links):** nối với span ở trace khác. [Trí nhớ]

## 5. Metric

Metric là số đo được tổng hợp theo thời gian. Các loại thường gặp: [Trí nhớ]

| Loại | Mô tả | Ví dụ |
|---|---|---|
| **Counter** | Chỉ tăng | Tổng số yêu cầu |
| **UpDownCounter** | Tăng hoặc giảm | Số kết nối đang mở |
| **Histogram** | Phân phối giá trị theo khoảng | Độ trễ yêu cầu |
| **Gauge** | Giá trị tại một thời điểm | Mức sử dụng CPU |

**Lưu ý:** tên loại metric trong SDK có thể khác nhau theo ngôn ngữ và phiên bản. [Suy luận]

## 6. Log

Log trong OpenTelemetry là bản ghi sự kiện có cấu trúc. Điểm quan trọng: log **có thể gắn với trace ID và span ID** để nối với trace tương ứng. Đây là cách chuyển từ "có lỗi" sang "lỗi xảy ra trong yêu cầu nào, span nào". [Trí nhớ]

## 7. Resource và attribute

- **Attribute:** cặp khóa-giá trị gắn vào span, metric, log. Ví dụ `http.request.method = "GET"`.
- **Resource:** thông tin mô tả nguồn phát telemetry: tên dịch vụ, phiên bản, môi trường, máy chủ. Resource được gắn chung cho mọi tín hiệu của một tiến trình. [Trí nhớ]

```
service.name = "dat-hang-service"
service.version = "1.4.0"
deployment.environment = "production"
```

## 8. Baggage

**Baggage** là dữ liệu ngữ cảnh truyền theo yêu cầu qua các dịch vụ, không phải telemetry. Ví dụ: ID người dùng hoặc mã khách hàng để dịch vụ phía sau dùng cho phân tích. [Trí nhớ]

**Cảnh báo:** baggage được gửi kèm mọi request nội bộ. Không đặt dữ liệu nhạy cảm, và cân nhắc kích thước. [Suy luận]

## 9. Semantic conventions

**Semantic conventions** là bộ quy ước tên thuộc tính và cấu trúc dữ liệu chung, để mọi ngôn ngữ và công cụ gọi cùng một thứ bằng cùng một tên. Ví dụ: thuộc tính phương thức HTTP là `http.request.method`. [Trí nhớ]

**Lợi ích:** truy vấn và dashboard hoạt động giống nhau giữa các dịch vụ viết bằng ngôn ngữ khác nhau. [Suy luận]

## 10. Propagation: truyền context qua dịch vụ

Để trace nối được qua nhiều dịch vụ, context phải được gửi kèm request. OpenTelemetry dùng chuẩn **W3C Trace Context** qua header `traceparent`. [Trí nhớ]

```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
             │  │                                │                  │
             │  trace-id (32 ký tự hex)          span-id (16 hex)   cờ lấy mẫu
             phiên bản
```

**Ý nghĩa:** dịch vụ nhận đọc header, tạo span con có cùng trace ID. Nếu thiếu propagation, mỗi dịch vụ tạo trace riêng, không nối được. [Suy luận]

---

# PHẦN C. KIẾN TRÚC

## 11. API, SDK, và instrumentation

```
┌──────────────────────────────────────────┐
│  Ứng dụng của bạn                        │
│   ↓ gọi API (đo, tạo span)              │
│  API   (giao diện, không làm gì)         │
│   ↓                                      │
│  SDK   (xử lý, lấy mẫu, gom lô, xuất)    │
│   ↓                                      │
│  Exporter (OTLP, ...)                    │
└──────────────────┬───────────────────────┘
                   ▼
            Collector hoặc backend
```

- **API:** giao diện để ứng dụng gọi. Nếu không có SDK, API không làm gì (no-op). Điều này cho phép thư viện dùng API mà không ép người dùng cài SDK. [Trí nhớ]
- **SDK:** triển khai thực tế: tạo span, lấy mẫu, gom lô, và gửi đi.
- **Instrumentation:** mã đo lường. Có thể do tác giả framework tích hợp sẵn, hoặc do bạn viết thêm.

## 12. OTLP: giao thức truyền dữ liệu

**OTLP (OpenTelemetry Protocol)** là giao thức chuẩn để truyền trace, metric, log. Hỗ trợ gRPC và HTTP/protobuf (và JSON với HTTP). [Trí nhớ]

**Cổng mặc định** thường gặp: [Trí nhớ]
- `4317`: OTLP qua gRPC.
- `4318`: OTLP qua HTTP.

## 13. Collector: receiver, processor, exporter, pipeline

Trang kiến trúc của Collector mô tả: [Đã kiểm chứng]

**Collector** là một chương trình có thể nhận telemetry, xử lý, và gửi đến một hoặc nhiều backend.

```
Receiver ──► Processor ──► Processor ──► Exporter
(nhận)       (xử lý)       (xử lý)        (gửi)
```

| Thành phần | Vai trò | Ví dụ |
|---|---|---|
| **Receiver** | Nhận dữ liệu (đẩy vào cổng mạng) hoặc kéo dữ liệu (scraper) | `otlp`, `prometheus` |
| **Processor** | Sửa, lọc, hoặc tạo dữ liệu, chạy theo thứ tự | `batch`, `memory_limiter`, `attributes` |
| **Exporter** | Gửi đến đích | `otlp`, `debug` |
| **Pipeline** | Chuỗi receiver → processor → exporter cho một loại tín hiệu | `traces`, `metrics`, `logs` |

**Quy tắc quan trọng:**
- Mỗi pipeline xử lý **một loại tín hiệu**. Các thành phần trong đó phải hỗ trợ loại đó.
- Một receiver có thể gửi dữ liệu cho nhiều pipeline. Nếu một processor trong một pipeline bị chặn, các pipeline dùng chung receiver cũng bị ảnh hưởng. [Đã kiểm chứng]
- Mỗi pipeline có instance processor riêng, nên trạng thái processor không dùng chung. [Đã kiểm chứng]

**Connector và extension:** trang đã đọc chưa mô tả. Cần tài liệu riêng. [Chưa xác nhận]

## 14. Triển khai: agent và gateway

Hai mẫu triển khai được tài liệu mô tả: [Đã kiểm chứng]

| Mẫu | Vị trí | Ưu điểm | Nhược điểm |
|---|---|---|---|
| **Agent** | Chạy cùng ứng dụng: daemon trên VM, sidecar, DaemonSet trên Kubernetes | Giảm việc xuất trực tiếp từ thư viện; có thể đổi backend không cần triển khai lại ứng dụng | Quản lý nhiều instance |
| **Gateway** | Một điểm tập trung nhận dữ liệu từ agent hoặc ứng dụng, rồi chuyển đến backend | Quản lý tập trung, dễ áp dụng lấy mẫu và lọc | Điểm nghẽn nếu không scale |

**Thường kết hợp:** agent ở mỗi node, gateway ở tầng trung tâm. [Suy luận]

---

# PHẦN D. THỰC HÀNH: INSTRUMENTATION

## 15. Tự động (auto) và thủ công (manual) instrumentation

- **Tự động:** agent hoặc gói tích hợp sẵn đo các framework phổ biến (HTTP, cơ sở dữ liệu) mà không cần sửa mã. Phù hợp để bắt đầu nhanh. [Trí nhớ]
- **Thủ công:** bạn viết mã tạo span và metric cho logic nghiệp vụ. Cần khi muốn đo thao tác cụ thể. [Trí nhớ]

**Nguyên tắc:** bắt đầu tự động để có bức tranh tổng thể, rồi thêm thủ công cho các điểm nghiệp vụ quan trọng. [Suy luận]

## 16. Tạo span trong Python

Các ví dụ dưới dùng gói `opentelemetry-api` và `opentelemetry-sdk`. Tên lớp có thể thay đổi theo phiên bản, kiểm tra tài liệu Python SDK.

```python
from opentelemetry import trace                                   # import API tạo tracer
from opentelemetry.sdk.trace import TracerProvider                # SDK: nhà cung cấp tracer
from opentelemetry.sdk.trace.export import BatchSpanProcessor     # gom span thành lô rồi gửi
from opentelemetry.sdk.trace.export import ConsoleSpanExporter    # xuất span ra màn hình (để thử)

provider = TracerProvider()                                       # tạo nhà cung cấp tracer
provider.add_span_processor(                                      # đăng ký bộ xử lý span
    BatchSpanProcessor(ConsoleSpanExporter())                     # gom lô, rồi gửi bằng exporter in ra console
)
trace.set_tracer_provider(provider)                               # đặt làm nhà cung cấp toàn cục

tracer = trace.get_tracer("dat-hang")                             # lấy tracer có tên "dat-hang"

def dat_hang(ma_san_pham: str) -> None:                           # hàm nghiệp vụ
    with tracer.start_as_current_span("dat-hang") as span:        # tạo span mới, là span hiện tại trong khối
        span.set_attribute("san_pham.ma", ma_san_pham)            # gắn thuộc tính vào span
        kiem_tra_ton_kho(ma_san_pham)                             # gọi hàm con (sẽ tạo span con nếu có instrumentation)
        span.add_event("da-kiem-tra-ton-kho")                     # ghi một mốc sự kiện trong span

def kiem_tra_ton_kho(ma: str) -> None:
    with tracer.start_as_current_span("kiem-tra-ton-kho"):        # span con: tự động nối với span cha
        pass                                                      # giả lập công việc

dat_hang("SP-7")
```

| Cú pháp | Ý nghĩa |
|---|---|
| `trace.get_tracer("tên")` | Lấy tracer, tên thường là tên module hoặc dịch vụ |
| `with tracer.start_as_current_span("tên") as span:` | Tạo span, đặt làm span hiện tại; khi thoát khối, span tự kết thúc |
| `span.set_attribute(khóa, giá_trị)` | Gắn cặp khóa-giá trị vào span |
| `span.add_event("tên")` | Ghi mốc thời gian trong span |
| `BatchSpanProcessor` | Gom span thành lô để giảm số lần gửi |
| `ConsoleSpanExporter` | Xuất span ra console, dùng để kiểm tra khi học |

**Quan sát:** span con được tự động gắn vào span cha vì `start_as_current_span` đặt span hiện tại vào context.

**Ghi nhận lỗi:** nếu có ngoại lệ trong khối `with`, span thường tự ghi trạng thái lỗi. Có thể ghi thủ công bằng `span.record_exception(e)` và `span.set_status(...)`. [Trí nhớ; kiểm tra API]

## 17. Metric trong Python

```python
from opentelemetry import metrics                                  # import API metric
from opentelemetry.sdk.metrics import MeterProvider                # SDK: nhà cung cấp meter
from opentelemetry.sdk.metrics.export import (                     # các lớp để xuất metric
    ConsoleMetricExporter,
    PeriodicExportingMetricReader,
)

reader = PeriodicExportingMetricReader(ConsoleMetricExporter())    # định kỳ đọc và xuất metric
metrics.set_meter_provider(MeterProvider(metric_readers=[reader])) # đặt nhà cung cấp meter toàn cục

meter = metrics.get_meter("dat-hang")                              # lấy meter theo tên
don_hang = meter.create_counter(                                   # tạo counter (chỉ tăng)
    name="don_hang.tong",                                          # tên metric
    unit="1",                                                      # đơn vị: "1" nghĩa là đếm
    description="Tổng số đơn hàng",                                # mô tả
)

don_hang.add(1, {"khu_vuc": "bac"})                                # tăng 1, kèm thuộc tính khu_vuc
```

| Cú pháp | Ý nghĩa |
|---|---|
| `create_counter(...)` | Tạo counter, tăng dần |
| `.add(giá_trị, attributes)` | Tăng giá trị, kèm thuộc tính |
| `PeriodicExportingMetricReader` | Định kỳ gửi metric, mặc định vài chục giây |
| `unit="1"` | Đơn vị đếm, không có đơn vị vật lý |

**Lưu ý về thuộc tính:** mỗi tổ hợp giá trị thuộc tính tạo một chuỗi thời gian riêng. Tránh dùng ID người dùng hoặc URL đầy đủ làm thuộc tính metric. Xem mục 25.

## 18. Log và liên kết với trace

```python
import logging                                                     # thư viện logging chuẩn của Python
from opentelemetry import trace                                    # để lấy span hiện tại

logger = logging.getLogger("dat-hang")                             # logger có tên dat-hang
logger.setLevel(logging.INFO)                                      # chỉ ghi INFO trở lên

def xu_ly():
    span = trace.get_current_span()                                # lấy span đang chạy
    ctx = span.get_span_context()                                  # lấy thông tin context của span
    logger.info(                                                   # ghi log
        "xử lý đơn hàng",
        extra={"trace_id": format(ctx.trace_id, "032x")},          # gắn trace ID vào bản ghi (định dạng hex 32 ký tự)
    )
```

**Thực tế:** thường dùng thư viện tích hợp (như instrumentation cho `logging`) để tự động gắn trace ID vào bản ghi. [Trí nhớ]

| Cú pháp | Ý nghĩa |
|---|---|
| `trace.get_current_span()` | Lấy span đang hoạt động trong context hiện tại |
| `get_span_context()` | Lấy trace ID và span ID |
| `format(x, "032x")` | Định dạng số nguyên thành hex, đủ 32 ký tự |
| `extra={...}` | Thêm trường tùy chỉnh vào bản ghi log |

## 19. Sampling

**Sampling** quyết định giữ lại bao nhiêu trace, để giảm chi phí. [Trí nhớ]

| Kiểu | Cách hoạt động |
|---|---|
| **Head sampling** | Quyết định ngay đầu trace, dựa trên tỉ lệ hoặc thuộc tính |
| **Tail sampling** | Quyết định sau khi trace hoàn tất, ví dụ giữ hết trace lỗi. Thường làm ở Collector |

```python
from opentelemetry.sdk.trace.sampling import ParentBased, TraceIdRatioBased

sampler = ParentBased(TraceIdRatioBased(0.1))     # lấy 10% trace gốc, theo quyết định của cha nếu có
provider = TracerProvider(sampler=sampler)        # áp dụng sampler cho provider
```

| Cú pháp | Ý nghĩa |
|---|---|
| `TraceIdRatioBased(0.1)` | Lấy mẫu theo tỉ lệ 10% |
| `ParentBased(...)` | Nếu span cha đã quyết định, span con theo quyết định đó |

**Lưu ý:** head sampling đơn giản nhưng có thể bỏ mất trace lỗi. Tail sampling giữ được trace lỗi, nhưng tốn tài nguyên ở Collector. [Suy luận]

---

# PHẦN E. THỰC HÀNH: COLLECTOR VÀ CẤU HÌNH

## 20. Cấu hình Collector

```yaml
receivers:                       # phần khai báo receiver
  otlp:                          # receiver tên otlp
    protocols:                   # giao thức được nhận
      grpc:                      # nhận OTLP qua gRPC
        endpoint: 0.0.0.0:4317   # lắng nghe mọi địa chỉ, cổng 4317
      http:                      # nhận OTLP qua HTTP
        endpoint: 0.0.0.0:4318   # cổng 4318

processors:                      # phần khai báo processor
  batch:                         # gom lô trước khi gửi
    timeout: 5s                  # gửi sau tối đa 5 giây
    send_batch_size: 512         # hoặc khi đủ 512 bản ghi
  memory_limiter:                # giới hạn bộ nhớ
    check_interval: 1s           # kiểm tra mỗi giây
    limit_percentage: 80         # ngưỡng 80% bộ nhớ

exporters:                       # phần khai báo exporter
  debug:                         # xuất ra log của Collector (để thử)
    verbosity: detailed          # mức chi tiết
  otlp:                          # gửi đến backend qua OTLP
    endpoint: backend.example.com:4317   # địa chỉ backend (ví dụ)
    tls:
      insecure: false            # bật TLS, không tắt

service:                         # phần nối các thành phần
  pipelines:                     # danh sách pipeline
    traces:                      # pipeline cho trace
      receivers: [otlp]          # nhận từ otlp
      processors: [memory_limiter, batch]   # chạy theo thứ tự này
      exporters: [debug, otlp]   # gửi đến cả debug và otlp
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
```

| Cú pháp | Ý nghĩa |
|---|---|
| `receivers:`, `processors:`, `exporters:` | Ba khối khai báo thành phần |
| `0.0.0.0:4317` | Lắng nghe mọi giao diện, cổng 4317 |
| `[a, b]` | Danh sách trong YAML |
| `service.pipelines` | Nối thành phần thành pipeline |
| Thứ tự trong `processors` | Quyết định thứ tự xử lý |

**Lưu ý:** thứ tự processor có ảnh hưởng. `memory_limiter` nên đứng đầu để chặn sớm. [Trí nhớ]

## 21. Chạy Collector bằng Docker

```sh
docker run --rm \                                        # --rm: xóa container khi dừng
  -p 4317:4317 -p 4318:4318 \                            # ánh xạ cổng máy chủ với container
  -v "$(pwd)/otel-config.yaml:/etc/otelcol/config.yaml" \  # gắn tệp cấu hình vào container
  otel/opentelemetry-collector-contrib:latest \          # ảnh chính thức (kiểm tra phiên bản cố định trước khi dùng)
  --config /etc/otelcol/config.yaml                      # chỉ định đường dẫn cấu hình
```

**Lưu ý:** `:latest` tiện để thử, nhưng khi triển khai nên ghim phiên bản cụ thể. [Suy luận]

## 22. Kiểm tra và gỡ lỗi

- **Exporter `debug`:** in span hoặc metric ra log của Collector, để xác nhận dữ liệu đến và đi đúng.
- **Kiểm tra cổng:** `nc -vz localhost 4317` để xem Collector có lắng nghe không.
- **Log của Collector:** xem lỗi cấu hình khi khởi động.
- **Kiểm tra cấu hình:** Collector có lệnh kiểm tra cú pháp và pipeline (kiểm tra tài liệu phiên bản đang dùng). [Trí nhớ]

---

# PHẦN F. KIỂM THỬ VÀ TỰ ĐỘNG HÓA

## 23. Kiểm thử instrumentation

Dùng exporter trong bộ nhớ để kiểm tra span được tạo đúng:

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor
from opentelemetry.sdk.trace.export.in_memory_span_exporter import InMemorySpanExporter

exporter = InMemorySpanExporter()                                # xuất vào bộ nhớ thay vì mạng
provider = TracerProvider()                                      # tạo provider cho test
provider.add_span_processor(SimpleSpanProcessor(exporter))       # gửi ngay, không gom lô
tracer = provider.get_tracer("test")

with tracer.start_as_current_span("kiem-thu") as span:           # tạo span
    span.set_attribute("khoa", "gia_tri")

spans = exporter.get_finished_spans()                            # lấy các span đã kết thúc
assert len(spans) == 1                                           # kiểm tra có đúng một span
assert spans[0].name == "kiem-thu"                               # kiểm tra tên span
assert spans[0].attributes["khoa"] == "gia_tri"                  # kiểm tra thuộc tính
```

| Cú pháp | Ý nghĩa |
|---|---|
| `InMemorySpanExporter` | Lưu span trong bộ nhớ để kiểm tra |
| `SimpleSpanProcessor` | Gửi ngay từng span, tiện cho test (không gom lô) |
| `get_finished_spans()` | Lấy danh sách span đã kết thúc |

**Ghi chú:** đường dẫn module của `InMemorySpanExporter` có thể khác theo phiên bản. Kiểm tra tài liệu SDK. [Trí nhớ]

## 24. Kiểm tra cấu hình trong CI

```yaml
- name: kiem tra cau hinh collector          # tên bước
  run: |                                     # khối lệnh nhiều dòng
    docker run --rm \
      -v "$PWD/otel-config.yaml:/etc/otelcol/config.yaml" \
      otel/opentelemetry-collector-contrib:0.0.0 \   # ghim phiên bản thật khi dùng (kiểm tra danh sách phiên bản)
      validate --config /etc/otelcol/config.yaml     # lệnh kiểm tra cấu hình
```

[Trí nhớ; lệnh `validate` và tag phiên bản cần kiểm tra]

---

# PHẦN G. VẬN HÀNH VÀ AN TOÀN

## 25. Chi phí, cardinality, và dữ liệu nhạy cảm

**Cardinality** là số tổ hợp giá trị duy nhất của thuộc tính. Cardinality cao làm tăng chi phí lưu trữ và truy vấn. [Trí nhớ]

| Nên | Không nên |
|---|---|
| `http.route = "/san-pham/{id}"` (template) | `http.url = "/san-pham/123456"` (chứa ID) |
| `khu_vuc = "bac"` | `user.id = "u-88213"` làm nhãn metric |
| Trạng thái: `ok`, `error` | Nội dung tự do từ người dùng |

**Dữ liệu nhạy cảm:** span và log có thể vô tình chứa mật khẩu, token, số thẻ, dữ liệu cá nhân. Cần:
- Không đưa dữ liệu nhạy cảm vào thuộc tính.
- Dùng processor để xóa hoặc che trường nhạy cảm ở Collector. [Trí nhớ]
- Kiểm soát quyền truy cập backend. [Suy luận]

## 26. Triển khai trên Kubernetes

- **DaemonSet:** chạy Collector trên mỗi node, nhận telemetry từ pod cùng node. [Trí nhớ]
- **Deployment:** chạy gateway tập trung, scale theo tải. [Trí nhớ]
- **Operator:** OpenTelemetry Operator quản lý Collector và tự động gắn instrumentation. [Trí nhớ]

```yaml
apiVersion: apps/v1                           # phiên bản API của Deployment
kind: Deployment                              # loại tài nguyên
metadata:
  name: otel-gateway                          # tên Deployment
spec:
  replicas: 2                                 # chạy 2 bản sao
  selector:
    matchLabels:
      app: otel-gateway                       # chọn pod có nhãn app=otel-gateway
  template:
    metadata:
      labels:
        app: otel-gateway
    spec:
      containers:
        - name: collector
          image: otel/opentelemetry-collector-contrib:0.0.0   # ghim phiên bản thật khi dùng
          ports:
            - containerPort: 4317             # cổng OTLP gRPC
```

[Trí nhớ; phiên bản ảnh và cấu hình cần kiểm tra]

---

# PHẦN H. VAI TRÒ TRONG HỆ SINH THÁI

## 27. So sánh với các giải pháp khác

| Tiêu chí | OpenTelemetry | Giải pháp độc quyền | Prometheus |
|---|---|---|---|
| Phạm vi | Trace, metric, log (và thêm) | Thường đầy đủ, khép kín | Chủ yếu metric |
| Khóa nhà cung cấp | Thấp, do chuẩn chung | Cao | Thấp |
| Mô hình | Push (OTLP) và pull | Thường push qua agent riêng | Pull (scrape) |
| Backend | Nhiều lựa chọn (mã nguồn mở và thương mại) | Thường là backend riêng | Prometheus và tương thích |

[Suy luận và kiến thức nền; so sánh khái quát]

**Ghi chú:** OpenTelemetry không phải là backend lưu trữ. Nó chuẩn hóa cách thu thập và gửi. Cần một backend để lưu và truy vấn. [Đã kiểm chứng: mô tả Collector là công cụ nhận, xử lý, và xuất]

## 28. Các điểm cần cẩn trọng

- **Trạng thái phiên bản:** tín hiệu và SDK ở các mức khác nhau. Kiểm tra trạng thái từng thành phần trước khi phụ thuộc. [Đã kiểm chứng]
- **Chi phí dữ liệu:** trace và metric lớn tốn tiền lưu trữ. Dùng sampling và giới hạn cardinality. [Suy luận]
- **Hiệu năng ứng dụng:** instrumentation tốn CPU và bộ nhớ. Đo trước và sau khi bật. [Suy luận]
- **Dữ liệu nhạy cảm:** lọc ở nguồn hoặc Collector. [Trí nhớ]
- **Ghi đè tên và thuộc tính:** tuân theo semantic conventions để dashboard dùng được. [Suy luận]
- **Tài liệu thay đổi nhanh:** nhiều tên API và cấu hình thay đổi theo phiên bản. Luôn đối chiếu tài liệu chính thức. [Suy luận]

---

# PHẦN I. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Telemetry | Dữ liệu đo từ hệ thống đang chạy |
| Observability | Khả năng hiểu trạng thái hệ thống qua dữ liệu bên ngoài |
| Trace | Đường đi của một yêu cầu qua nhiều span |
| Span | Một đơn vị công việc có tên và thời gian |
| Trace ID / Span ID | Định danh duy nhất của trace / span |
| Context | Thông tin nối span với trace |
| Metric | Số đo tổng hợp theo thời gian |
| Counter / Histogram / Gauge | Các loại metric |
| Log | Bản ghi sự kiện có cấu trúc |
| Resource | Thông tin mô tả nguồn phát telemetry |
| Attribute | Cặp khóa-giá trị gắn vào telemetry |
| Baggage | Dữ liệu ngữ cảnh truyền qua các dịch vụ |
| Semantic conventions | Quy ước tên và cấu trúc chung |
| Propagation | Truyền context qua các dịch vụ |
| W3C Trace Context | Chuẩn header `traceparent` |
| API / SDK | Giao diện / triển khai |
| Instrumentation | Mã đo lường (tự động hoặc thủ công) |
| OTLP | Giao thức truyền dữ liệu của OpenTelemetry |
| Collector | Chương trình nhận, xử lý, xuất telemetry |
| Receiver / Processor / Exporter | Ba loại thành phần trong pipeline |
| Pipeline | Chuỗi thành phần cho một loại tín hiệu |
| Agent / Gateway | Hai mẫu triển khai Collector |
| Sampling | Quyết định giữ lại bao nhiêu trace |
| Head / Tail sampling | Lấy mẫu lúc đầu trace / sau khi trace hoàn tất |
| Cardinality | Số giá trị duy nhất của một thuộc tính |

---

# PHẦN J. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## J.1. Nguồn đã đối chiếu trong lần này

- **OpenTelemetry, Signals**: https://opentelemetry.io/docs/concepts/signals/ [đã đọc. Ghi nhận: traces, metrics, logs, baggage được hỗ trợ; profiles và events đang phát triển]
- **OpenTelemetry Collector, Architecture**: https://opentelemetry.io/docs/collector/architecture/ [đã đọc. Ghi nhận: pipeline, receiver, processor, exporter, quy tắc một loại tín hiệu mỗi pipeline, mẫu agent và gateway. Không có phiên bản]
- **OpenTelemetry Specification, Versioning and Stability**: https://opentelemetry.io/docs/specs/otel/versioning-and-stability/ [đã đọc. Ghi nhận: bốn mức Development, Stable, Deprecated, Removed và ràng buộc cho API, SDK, semantic conventions]

## J.2. Nguồn chưa đối chiếu

- Tài liệu SDK Python, Java, Go, JavaScript về API cụ thể (tên lớp, tham số).
- Tài liệu semantic conventions (tên thuộc tính HTTP, database, messaging).
- Tài liệu Collector về connector, extension, và danh sách processor, exporter.
- Đặc tả W3C Trace Context.
- Tài liệu OpenTelemetry Operator.

## J.3. Điểm cần xác minh

1. **Tên API trong các ví dụ Python** (`TracerProvider`, `BatchSpanProcessor`, `PeriodicExportingMetricReader`, `ParentBased`, `InMemorySpanExporter`): theo kiến thức nền, cần kiểm tra tài liệu Python SDK đúng phiên bản.
2. **Phiên bản Collector và ảnh Docker:** `0.0.0` và `latest` chỉ là ví dụ. Cần chọn phiên bản cụ thể từ kho phát hành.
3. **Lệnh `validate`** của Collector và cách kiểm tra cấu hình: cần xác nhận theo tài liệu.
4. **Cấu hình processor** (`memory_limiter`, `batch`) và tên tham số: kiến thức nền.
5. **Tên đường dẫn** của instrumentation tự động cho từng ngôn ngữ: thay đổi theo thời gian.
6. **Trạng thái profiles và events:** tài liệu đã đọc ghi là đang phát triển, cần kiểm tra cập nhật.
7. **Các ví dụ mã chưa được chạy thử.** Nên thử trên máy trước khi dùng.

---

**Tóm tắt để tra nhanh:**
- **OpenTelemetry** = chuẩn mở để thu thập và gửi trace, metric, log, không kèm backend.
- **Trace** gồm các **span** nối với nhau qua **context**; **propagation** giữ trace qua các dịch vụ.
- **API** (giao diện) tách khỏi **SDK** (triển khai); **OTLP** là giao thức truyền.
- **Collector** có **receiver → processor → exporter** trong **pipeline**; triển khai theo mẫu **agent** hoặc **gateway**.
- **Sampling** và **cardinality** là hai yếu tố kiểm soát chi phí.
- **Cẩn trọng:** kiểm tra độ ổn định của từng thành phần, tránh dữ liệu nhạy cảm trong telemetry, và đối chiếu tên API với đúng phiên bản SDK.
