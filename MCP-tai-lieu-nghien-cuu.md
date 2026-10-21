# Model Context Protocol (MCP): Tài liệu tham khảo chuyên sâu

> **Viết cho:** người muốn hiểu MCP ở mức kỹ thuật và nghiên cứu, đọc tiếng Việt, cần đối chiếu với đặc tả gốc.
> **Thay thế:** file `MCP-tai-lieu.md` (bản tổng quan). File này đính chính một số nội dung đã lỗi thời, đặc biệt phần handshake.
> **Phạm vi:** đặc tả các phiên bản 2024-11-05 đến 2026-07-28. Mục 2 trình bày lịch sử phiên bản.
> **Cách đọc nhãn:** `[Đã kiểm chứng]` = khớp với changelog chính thức đã tra cứu; `[Trí nhớ]` = mô tả từ kiến thức nền, nên đối chiếu lại với đặc tả trước khi dùng cho việc chính thức.

---

## Mục lục

- Phần A. Nền tảng
  - 1. Định nghĩa và phạm vi
  - 2. Lịch sử phiên bản
  - 3. Bối cảnh: vì sao MCP ra đời
  - 4. So sánh với các chuẩn liên quan
- Phần B. Mô hình kiến trúc
  - 5. Vai trò: Host, Client, Server
  - 6. Kiến trúc SDK theo lớp
  - 7. Mô hình ngữ cảnh: ai quyết định gì
- Phần C. Giao thức lõi
  - 8. JSON-RPC 2.0 trong MCP
  - 9. Vòng đời và đàm phán (theo từng phiên bản)
  - 10. Capabilities
  - 11. Các primitive phía server: Tools, Resources, Prompts
  - 12. Các primitive phía client: Sampling, Elicitation, Roots
  - 13. Mẫu tương tác server → client: MRTR và subscriptions
  - 14. Tiện ích: progress, cancellation, logging, pagination, cache
  - 15. Xử lý lỗi
  - 16. Tasks (tác vụ dài)
- Phần D. Transport
  - 17. stdio
  - 18. Streamable HTTP
  - 19. Các transport khác và tương lai
- Phần E. Ủy quyền và bảo mật
  - 20. Ủy quyền (Authorization)
  - 21. Mô hình đe dọa
  - 22. Biện pháp phòng vệ và nguyên tắc thiết kế
- Phần F. Vận hành
  - 23. Chi phí ngữ cảnh
  - 24. Quan sát, gỡ lỗi, kiểm thử
  - 25. Triển khai và mở rộng quy mô
  - 26. Thiết kế tool tốt
- Phần G. Hệ sinh thái và thực hành
  - 27. Cấu hình ở các host
  - 28. Ví dụ server và client
  - 29. Hệ sinh thái
- Phần H. Phụ lục
  - A. Bảng method đầy đủ theo phiên bản
  - B. Bảng mã lỗi
  - C. Glossary song ngữ Anh–Việt
  - D. Danh mục nguồn tham khảo
  - E. Câu hỏi mở và điểm cần kiểm chứng

---

# PHẦN A. NỀN TẢNG

## 1. Định nghĩa và phạm vi

**MCP (Model Context Protocol)** là giao thức mở để một ứng dụng dùng mô hình ngôn ngữ (host) truy cập có kiểm soát vào **công cụ (tools)**, **dữ liệu (resources)** và **mẫu lệnh (prompts)** do các chương trình độc lập (server) cung cấp. Đặc tả định nghĩa thông điệp, vòng đời, transport, các primitive và yêu cầu bảo mật. Đặc tả **không** quy định cách mô hình được huấn luyện, cách host chọn công cụ, hay giao diện người dùng cụ thể.

Ba điểm làm nên đặc trưng của MCP:
1. **Tách rời (decoupling):** server viết một lần, dùng được ở mọi host tuân thủ giao thức.
2. **Khám phá động (dynamic discovery):** host hỏi server có những khả năng gì, không cần biết trước khi biên dịch.
3. **Kiểm soát bởi con người:** đặc tả yêu cầu host để người dùng duyệt các thao tác có tác dụng phụ.

## 2. Lịch sử phiên bản

Mỗi phiên bản đặc tả có định danh là một ngày. Bảng dưới tổng hợp thay đổi chính.

| Phiên bản | Điểm chính | Nhãn |
|---|---|---|
| **2024-11-05** | Bản đầu công khai. Transport stdio và HTTP+SSE. Tools, Resources, Prompts, Sampling, Roots. | [Trí nhớ] |
| **2025-03-26** | Khung ủy quyền dựa trên **OAuth 2.1**. Thay HTTP+SSE bằng **Streamable HTTP**. Thêm JSON-RPC batching (bị bỏ ở 2025-06-18). Thêm **tool annotations** (`readOnlyHint`, `destructiveHint`...). Thêm audio, `message` trong progress, capability `completions`. | [Đã kiểm chứng] |
| **2025-06-18** | Bỏ JSON-RPC batching. Thêm **structured tool output**. Phân loại server là **OAuth Resource Server**, thêm protected resource metadata. Client **phải** dùng RFC 8707 (resource indicators). Thêm **elicitation** và **resource link** trong kết quả tool. Header `MCP-Protocol-Version` bắt buộc trên HTTP. Thêm trang bảo mật riêng. Thêm `title` cho hiển thị, `name` dùng như định danh. | [Đã kiểm chứng] |
| **2025-11-25** | **URL mode elicitation** (SEP-1036). **Tasks** dạng thử nghiệm (SEP-1686). **Sampling có tool** qua `tools` và `toolChoice` (SEP-1577). **Client ID Metadata Documents** được khuyến nghị làm cơ chế đăng ký client (SEP-991). Khám phá authorization server theo **OpenID Connect Discovery** (PR #797). Tăng quyền dần qua `WWW-Authenticate` (SEP-835). **Icons** cho tool, resource, prompt (SEP-973). Enum có tiêu đề và multi-select (SEP-1330). Default cho mọi kiểu primitive (SEP-1034). **JSON Schema 2020-12** là mặc định (SEP-1613). Lỗi đầu vào là lỗi thực thi tool, không phải lỗi giao thức (SEP-1303). Hướng dẫn tên tool (SEP-986). Khám phá tài nguyên theo RFC 9728 (SEP-985). Quy tắc SSE polling (SEP-1699). | [Đã kiểm chứng] |
| **2026-07-28** | **Thay đổi lớn:** bỏ handshake `initialize`, chuyển sang **stateless** (mỗi request tự mang phiên bản và capabilities trong `_meta`), thêm `server/discover`, thay GET/subscribe bằng `subscriptions/listen`, bỏ `ping`, `logging/setLevel`, `notifications/roots/list_changed`, bỏ resumability của SSE, thêm **MRTR** (Multi Round-Trip Requests) thay cho request ngược từ server, tasks chuyển thành extension, deprecated Roots, Sampling và Logging. | [Đã kiểm chứng] |

**Lưu ý quan trọng khi đọc tài liệu:** nhiều hướng dẫn trên mạng (và phần lớn bản tổng quan cũ) mô tả handshake `initialize` → `notifications/initialized`. Đó là mô hình của các phiên bản đến 2025-11-25. Với 2026-07-28, handshake đã bị bỏ. Khi triển khai, hãy xác định phiên bản đặc tả nào mà SDK và server bạn dùng hỗ trợ.

Theo changelog 2026-07-28 [Đã kiểm chứng], các nhóm thay đổi chính là:
- **Stateless:** bỏ `Mcp-Session-Id`, bỏ `initialize`. Danh sách (`tools/list`, `resources/list`, `prompts/list`) không còn thay đổi theo từng kết nối. Trạng thái nếu cần được truyền qua handle do server cấp, đi như tham số tool thông thường.
- **Discovery:** `server/discover` là method bắt buộc với server, để công bố phiên bản, capabilities và danh tính.
- **Thông báo:** `subscriptions/listen` thay cho HTTP GET và `resources/subscribe`/`resources/unsubscribe`. Client chọn loại thông báo muốn nhận.
- **Kết quả có `resultType`:** `"complete"` hoặc `"input_required"`. Client phải coi kết quả thiếu trường này (từ server phiên bản cũ) là `"complete"`.
- **Cache:** kết quả list và read có `ttlMs` và `cacheScope` (`"public"` hoặc `"private"`).
- **Lỗi:** mã `-32002` (resource not found) đổi thành `-32602` (Invalid Params). Mã lỗi của server được phân vùng: `-32000` đến `-32019` do triển khai tự định nghĩa, `-32020` đến `-32099` dành cho đặc tả.
- **Schema:** `inputSchema`/`outputSchema` cho phép mọi từ khóa JSON Schema 2020-12. `structuredContent` cho phép mọi giá trị JSON.
- **Ủy quyền:** RFC 9207 (tham số `iss`), bắt buộc `application_type` khi đăng ký động, credential gắn với issuer. Deprecated: RFC 7591 (Dynamic Client Registration), ưu tiên Client ID Metadata Documents.

## 3. Bối cảnh: vì sao MCP ra đời

Trước MCP, tích hợp công cụ với LLM có ba đặc điểm gây khó:

1. **Tích hợp N×M:** `N` ứng dụng AI nhân `M` dịch vụ tạo ra `N×M` bộ kết nối. Mỗi bộ kết nối có xác thực, định dạng và cách mô tả riêng.
2. **Tool calling không chuẩn:** mỗi nhà cung cấp mô hình có định dạng định nghĩa và gọi hàm riêng. Cùng một công cụ phải viết lại nhiều lần.
3. **Ngữ cảnh rời rạc:** dữ liệu và công cụ không được đưa vào mô hình theo một quy ước chung, nên khó kiểm soát việc gì được gửi cho mô hình và ai đã đồng ý.

MCP giải quyết bằng cách tách **giao thức** khỏi **mô hình** và khỏi **ứng dụng**. Công cụ được mô tả bằng JSON Schema, kết quả trả về theo cấu trúc chuẩn, và host là nơi đặt các điểm kiểm soát.

Nguồn cảm hứng trực tiếp là **LSP (Language Server Protocol)**: một server ngôn ngữ phục vụ nhiều editor qua một giao thức chung. MCP áp dụng cùng ý tưởng cho AI.

## 4. So sánh với các chuẩn liên quan

| Tiêu chí | MCP | Function calling (từng nhà cung cấp) | OpenAPI | LSP | A2A (Agent2Agent) |
|---|---|---|---|---|---|
| Mục tiêu | Kết nối AI với công cụ, dữ liệu, mẫu lệnh | Mô hình chọn hàm và sinh tham số | Mô tả API HTTP | Tính năng ngôn ngữ cho editor | Giao tiếp giữa các agent |
| Định nghĩa | Tool có inputSchema/outputSchema | Tool có JSON Schema trong yêu cầu | Tài liệu API YAML/JSON | Method do LSP định nghĩa | Agent Card |
| Phát hiện | `tools/list` động | Định nghĩa gắn cứng trong yêu cầu | Tài liệu tĩnh | Khởi tạo, capabilities | Agent Card |
| Vai trò thực thi | Server thực thi, host điều phối | Ứng dụng tự thực thi | Dịch vụ HTTP thực thi | Server ngôn ngữ thực thi | Agent thực thi |
| Quan hệ | Dùng **bên trong** một ứng dụng AI | Tính năng của mô hình | Có thể được bọc thành MCP server | Nguồn cảm hứng cho MCP | Bổ sung cho MCP (agent ↔ agent) |

Một cách hiểu thực tế: MCP thường nằm **phía dưới** tool calling. Một ứng dụng dùng function calling của mô hình, còn định nghĩa và thực thi các hàm đó có thể đến từ MCP server.

---

# PHẦN B. MÔ HÌNH KIẾN TRÚC

## 5. Vai trò: Host, Client, Server

```
┌──────────────────────── HOST ─────────────────────────┐
│  Mô hình ngôn ngữ · Chính sách · Giao diện · Consent  │
│                                                       │
│   Client A ──1:1──► Server A  (stdio, cục bộ)         │
│   Client B ──1:1──► Server B  (Streamable HTTP)       │
│   Client C ──1:1──► Server C  (Streamable HTTP)       │
└───────────────────────────────────────────────────────┘
```

| Vai trò | Trách nhiệm | Không làm |
|---|---|---|
| **Host** | Chứa mô hình, tạo client, quyết định nội dung đưa vào ngữ cảnh, hỏi consent, áp chính sách bảo mật, lọc đầu ra | Không tự giao tiếp với server bằng thông điệp riêng; việc đó do client làm |
| **Client** | Một kết nối 1:1 đến một server, gửi request, nhận response và notification, quản lý timeout và phiên | Không quyết định nội dung nào vào mô hình |
| **Server** | Cung cấp tools, resources, prompts; thực thi khi được gọi; khai báo capabilities | Không biết các server khác; không thấy toàn bộ hội thoại |

**Nguyên tắc cô lập:** mỗi client chỉ biết một server. Host là điểm duy nhất có cái nhìn tổng thể, nên là nơi đặt chính sách liên server (ví dụ không cho server A đọc kết quả của server B).

## 6. Kiến trúc SDK theo lớp

SDK chính thức thường chia thành ba lớp. Cách chia này giúp thay transport mà không sửa logic nghiệp vụ.

```
┌───────────────────────────────┐
│ Lớp nghiệp vụ                 │  đăng ký tool/resource/prompt,
│ (Server / Client API)         │  handler, schema
├───────────────────────────────┤
│ Lớp giao thức                 │  định tuyến method, kiểm tra
│ (Protocol / Session)          │  schema, id request, timeout,
│                               │  hủy, progress
├───────────────────────────────┤
│ Lớp transport                 │  stdio, Streamable HTTP
│ (Transport)                   │  đóng gói/tách JSON-RPC
└───────────────────────────────┘
```

Với phiên bản 2026-07-28, lớp giao thức bớt phần session vì không còn `Mcp-Session-Id`. [Đã kiểm chứng]

## 7. Mô hình ngữ cảnh: ai quyết định gì

Một điểm dễ bị bỏ sót là **ai điều khiển từng primitive**. Đây là nền để hiểu bảo mật.

| Primitive | Ai khởi tạo lời gọi | Ai quyết định dùng | Ai duyệt |
|---|---|---|---|
| Tool | Mô hình (qua host) | Mô hình | Host, thường có xác nhận người dùng |
| Resource | Host hoặc người dùng | Host | Host |
| Prompt | Người dùng | Người dùng | Người dùng |
| Sampling | Server (đã bị deprecated 2026-07-28) | Host | Host |
| Elicitation | Server | Người dùng | Người dùng |
| Roots | Client khai báo (đã bị deprecated) | Server | Client |

---

# PHẦN C. GIAO THỨC LÕI

## 8. JSON-RPC 2.0 trong MCP

MCP trao đổi tin nhắn JSON-RPC 2.0 UTF-8. Có ba dạng:

| Dạng | Trường chính | Phản hồi? |
|---|---|---|
| Request | `jsonrpc`, `id`, `method`, `params` | Có, cùng `id` |
| Response | `jsonrpc`, `id`, `result` hoặc `error` | Không |
| Notification | `jsonrpc`, `method`, `params`; không có `id` | Không |

Quy tắc đặc thù:
- `id` không được tái sử dụng trong cùng một phiên đang mở (với phiên bản stateless, quy tắc áp dụng cho từng request).
- JSON-RPC batching (gửi nhiều request trong một mảng) đã được bỏ từ 2025-06-18. [Trí nhớ]
- Trường `_meta` có thể mang metadata mở rộng, và các khóa do MCP định nghĩa dùng tiền tố `io.modelcontextprotocol/`. [Đã kiểm chứng]

Ví dụ tool call (dạng chung, đúng với mọi phiên bản):

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": { "city": "Hà Nội" }
  }
}
```

## 9. Vòng đời và đàm phán (theo từng phiên bản)

### 9.1. Phiên bản đến 2025-11-25: có handshake

```
Client                                         Server
  │ initialize {protocolVersion, capabilities, clientInfo}
  │──────────────────────────────────────────────►│
  │ result {protocolVersion, capabilities, serverInfo}
  │◄──────────────────────────────────────────────│
  │ notifications/initialized
  │──────────────────────────────────────────────►│
  │            … phiên hoạt động …
```

Quy tắc: server chọn `protocolVersion` mà nó hỗ trợ. Nếu client không chấp nhận, client ngắt kết nối. Client chỉ được gửi request sau khi nhận `initialize` result và gửi `notifications/initialized`, ngoại trừ `ping`. [Trí nhớ]

### 9.2. Phiên bản 2026-07-28: stateless [Đã kiểm chứng]

- Không có `initialize` hay `notifications/initialized`.
- Mỗi request mang trong `_meta` các khóa: `io.modelcontextprotocol/protocolVersion`, `io.modelcontextprotocol/clientCapabilities`, và nên có `io.modelcontextprotocol/clientInfo`.
- Server nên trả `io.modelcontextprotocol/serverInfo` trong `_meta` của kết quả.
- Nếu phiên bản không khớp, server trả `UnsupportedProtocolVersionError`.
- Client có thể gọi `server/discover` trước để chọn phiên bản và biết capabilities. Trên stdio, đây cũng là cách kiểm tra tương thích ngược.

Hệ quả thiết kế: vì mỗi request độc lập, server không giữ "phiên" để nhớ capabilities. Trạng thái cần giữa các lần gọi phải được truyền tường minh (ví dụ handle do server cấp, đưa vào tham số tool).

Ví dụ khám phá (theo cấu trúc changelog, chi tiết trường có thể thay đổi, nên đối chiếu schema):

```json
{ "jsonrpc": "2.0", "id": 1, "method": "server/discover" }
```

### 9.3. Tắt kết nối

- **stdio:** client đóng stdin, chờ tiến trình thoát, nếu quá thời gian thì kết thúc tiến trình.
- **Streamable HTTP:** đóng kết nối HTTP; không còn session để hủy trong phiên bản stateless.

## 10. Capabilities

Capabilities là khả năng một bên khai báo để bên kia biết những method nào được dùng.

| Phía | Capability | Ý nghĩa |
|---|---|---|
| Server | `tools` | Hỗ trợ `tools/list` và `tools/call`. `listChanged` báo khi danh sách đổi |
| Server | `resources` | Hỗ trợ `resources/list`, `resources/read`. `subscribe` cho phép theo dõi thay đổi |
| Server | `prompts` | Hỗ trợ `prompts/list`, `prompts/get` |
| Server | `logging` | Gửi log (đã deprecated 2026-07-28) |
| Server | `completions` | Gợi ý giá trị tham số |
| Client | `sampling` | Chấp nhận `sampling/createMessage` (đã deprecated 2026-07-28) |
| Client | `elicitation` | Chấp nhận `elicitation/create`. Có chế độ form và URL |
| Client | `roots` | Cung cấp `roots/list` (đã deprecated 2026-07-28) |
| Cả hai | `extensions` | Khai báo extension ngoài lõi (thêm từ 2026-07-28) [Đã kiểm chứng] |

Nguyên tắc: bên nào không khai báo capability thì bên kia không được gọi method tương ứng.

## 11. Các primitive phía server

### 11.1. Tools

**Mục đích:** hành động hoặc truy vấn có thể có tác dụng phụ, do mô hình chọn.

**Định nghĩa một tool:**

| Trường | Bắt buộc | Mô tả |
|---|---|---|
| `name` | Có | Định danh duy nhất trong server |
| `title` | Không | Tên hiển thị cho người |
| `description` | Nên có | Mô tả cho mô hình. Ảnh hưởng trực tiếp đến việc chọn tool |
| `inputSchema` | Có | JSON Schema 2020-12 cho tham số |
| `outputSchema` | Không | JSON Schema cho `structuredContent` |
| `annotations` | Không | Gợi ý: `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint` |

**Lưu ý:** annotation chỉ là gợi ý từ server. Host không được coi chúng là bảo đảm, vì server có thể nói dối. [Trí nhớ]

**Kết quả tool:**
- `content`: mảng phần tử kiểu text, image, audio, `resource_link`, hoặc resource nhúng.
- `structuredContent`: dữ liệu có cấu trúc, khớp `outputSchema`.
- `isError`: `true` nếu lỗi thực thi tool. Khác với lỗi giao thức (xem mục 15).

**Ví dụ kết quả:**

```json
{
  "content": [{ "type": "text", "text": "Hà Nội: 28°C, có mây" }],
  "structuredContent": { "city": "Hà Nội", "tempC": 28, "sky": "có mây" },
  "isError": false
}
```

**Thứ tự tool:** từ 2026-07-28, server nên trả tool theo thứ tự xác định để client cache và tăng tỷ lệ trúng cache prompt của mô hình. [Đã kiểm chứng]

### 11.2. Resources

**Mục đích:** dữ liệu chỉ đọc, định danh bằng URI, do host quyết định đưa vào ngữ cảnh.

- `resources/list`: liệt kê resource cụ thể.
- `resources/templates/list`: liệt kê URI template (ví dụ `db://customers/{id}`).
- `resources/read`: đọc nội dung, trả `contents` có `uri`, `mimeType`, và `text` hoặc `blob` (base64).
- Theo dõi thay đổi: 2025-11-25 dùng `resources/subscribe` và thông báo `notifications/resources/updated`. Từ 2026-07-28, dùng `subscriptions/listen` với `resourceSubscriptions`. [Đã kiểm chứng]

**Lỗi resource không tồn tại:** từ 2026-07-28 dùng `-32602` (Invalid Params), trước đó là `-32002`. [Đã kiểm chứng]

### 11.3. Prompts

**Mục đích:** mẫu hội thoại có tham số, người dùng chọn (thường thành lệnh slash).

- `prompts/list`: liệt kê, mỗi prompt có `name`, `description`, `arguments`.
- `prompts/get`: trả về danh sách `messages` đã điền tham số.

Prompt không tự gọi tool. Host nhận message và đưa cho mô hình.

## 12. Các primitive phía client

Đây là nhóm yêu cầu server gửi ngược lại client. Phiên bản 2026-07-28 đánh dấu **Roots, Sampling, Logging là deprecated**, và thay cơ chế request ngược bằng MRTR (mục 13). [Đã kiểm chứng]

### 12.1. Sampling (deprecated 2026-07-28)

- Server xin host sinh văn bản bằng mô hình của host, qua `sampling/createMessage`.
- Host giữ quyền chọn mô hình, giới hạn token, duyệt nội dung và chi phí.
- Từ 2025-11-25, sampling có thể kèm tool (`tools` và `toolChoice`), để server nhờ mô hình chạy vòng lặp gọi tool. [Trí nhớ]
- Rủi ro: server có thể lợi dụng sampling để đốt tài nguyên hoặc chèn chỉ dẫn độc hại. Host cần giới hạn số lần và kiểm soát nội dung.
- Migration khuyến nghị theo changelog: tích hợp trực tiếp với API của nhà cung cấp mô hình. [Đã kiểm chứng]

### 12.2. Elicitation (hỏi người dùng)

- `elicitation/create` gửi câu hỏi và một JSON Schema phẳng mô tả form.
- Chế độ **form:** host hiện form, người dùng nhập, trả về `accept`, `decline` hoặc `cancel` kèm dữ liệu.
- Chế độ **URL** (từ 2025-11-25): dùng cho dữ liệu nhạy cảm như OAuth hay thanh toán. Host mở URL trong trình duyệt, dữ liệu không đi qua client hay mô hình. [Trí nhớ]
- Changelog 2026-07-28 bỏ `notifications/elicitation/complete` và trường `elicitationId` của URL mode. Server nào cần liên kết qua các lần thử lại thì tự mã hóa định danh vào `requestState`. [Đã kiểm chứng]

### 12.3. Roots (deprecated 2026-07-28)

- Client khai báo danh sách URI (thường là thư mục dự án) mà server được phép làm việc.
- `roots/list` và thông báo thay đổi roots. Từ 2026-07-28, `notifications/roots/list_changed` bị bỏ. [Đã kiểm chứng]
- Migration: truyền đường dẫn qua tham số tool, qua URI resource, hoặc qua cấu hình server. [Đã kiểm chứng]
- Lưu ý: roots là gợi ý về phạm vi, không phải ranh giới bảo mật. Server phải tự kiểm tra đường dẫn (xem mục 21).

## 13. Mẫu tương tác server → client

### 13.1. Vấn đề

Ở các phiên bản cũ, server gửi request ngược (`sampling/createMessage`, `elicitation/create`, `roots/list`) trên cùng kết nối, và chờ client trả lời. Cách này buộc server giữ trạng thái giữa các request và yêu cầu kết nối hai chiều ổn định.

### 13.2. MRTR: Multi Round-Trip Requests (2026-07-28) [Đã kiểm chứng]

Thay vì gửi request ngược, server trả về **`InputRequiredResult`**:

```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "result": {
    "resultType": "input_required",
    "inputRequests": [
      { "...": "yêu cầu thông tin bổ sung" }
    ],
    "requestState": "trạng thái do server mã hóa"
  }
}
```

Client thực hiện yêu cầu (hỏi người dùng, gọi mô hình...), rồi **gửi lại request gốc** kèm `inputResponses` và `requestState`. Server tiếp tục xử lý từ trạng thái đó.

Điểm lợi:
- Không cần kết nối hai chiều giữ lâu.
- Phù hợp với mô hình stateless.
- Mọi kết quả đều có `resultType` (`"complete"` hoặc `"input_required"`).

Đánh đổi: server phải tự tuần tự hóa trạng thái vào `requestState`, và phải xử lý request lặp lại một cách an toàn (xem idempotency, mục 14.5).

### 13.3. subscriptions/listen (2026-07-28) [Đã kiểm chứng]

Thay cho kênh HTTP GET và `resources/subscribe`:
- Client mở một request `subscriptions/listen` và server giữ luồng phản hồi dài.
- Client khai báo loại thông báo muốn nhận: `toolsListChanged`, `promptsListChanged`, `resourcesListChanged`, `resourceSubscriptions`.
- Server xác nhận và gắn `io.modelcontextprotocol/subscriptionId` vào mỗi thông báo.
- Thông báo gắn với một request (ví dụ `notifications/progress`) vẫn đi trên luồng của request đó, không đi trên `subscriptions/listen`.

## 14. Tiện ích

### 14.1. Progress

- Client gửi `progressToken` trong `_meta` của request.
- Server gửi `notifications/progress` kèm `progress`, và `total` nếu biết.
- Progress phải tăng dần.

### 14.2. Cancellation

- Client gửi `notifications/cancelled` kèm `requestId`.
- Server có thể bỏ qua nếu request đã xong. Không được gửi response cho request đã hủy.
- Không hủy được request khởi tạo (`initialize`) trong phiên bản có handshake. [Trí nhớ]

### 14.3. Logging

- Phiên bản đến 2025-11-25: `logging/setLevel` và `notifications/message`.
- Từ 2026-07-28: deprecated. Mức log được đặt theo từng request qua `io.modelcontextprotocol/logLevel` trong `_meta`. Server không được gửi `notifications/message` cho request không có trường này. [Đã kiểm chứng]
- Migration khuyến nghị: log ra `stderr` trên stdio, hoặc dùng OpenTelemetry. [Đã kiểm chứng]

### 14.4. Pagination

- Kết quả list có `nextCursor`. Client gửi lại với `cursor` để lấy trang tiếp.
- Cursor là chuỗi mờ; client không được tự suy đoán cấu trúc.

### 14.5. Cache và idempotency

- Từ 2026-07-28, kết quả `tools/list`, `prompts/list`, `resources/list`, `resources/read`, `resources/templates/list` có `ttlMs` (mili-giây) và `cacheScope` (`"public"` hoặc `"private"`). [Đã kiểm chứng]
- `ttlMs` là gợi ý về thời gian còn tươi, giúp giảm polling. `cacheScope` quyết định trung gian dùng chung có được cache không.
- **Idempotency:** vì MRTR có thể gửi lại request, và client có thể retry sau khi mất kết nối, tool có tác dụng phụ phải được thiết kế để an toàn khi lặp lại (dùng khóa idempotency, hoặc kiểm tra trước khi ghi).

### 14.6. Ping

- Phiên bản đến 2025-11-25: `ping` để kiểm tra kết nối.
- Từ 2026-07-28: bị bỏ. [Đã kiểm chứng]

## 15. Xử lý lỗi

Có hai tầng lỗi, và cần phân biệt rõ:

| Tầng | Cách báo | Ví dụ |
|---|---|---|
| **Lỗi giao thức** (JSON-RPC) | Trường `error` trong response | Method không tồn tại, tham số sai schema |
| **Lỗi thực thi tool** | `result` với `isError: true` | API bên ngoài trả 500, file không tồn tại |

**Lý do cần tách:** lỗi thực thi tool được trả cho **mô hình** để nó tự sửa (đổi tham số, thử cách khác). Lỗi giao thức thường là lỗi lập trình của client hoặc server.

**Mã lỗi JSON-RPC chuẩn:**

| Mã | Ý nghĩa |
|---|---|
| -32700 | Parse error: JSON sai cú pháp |
| -32600 | Invalid Request: thiếu trường bắt buộc |
| -32601 | Method not found |
| -32602 | Invalid params |
| -32603 | Internal error |
| -32000 đến -32099 | Lỗi do triển khai định nghĩa (2026-07-28 phân vùng lại, xem mục 2 và phụ lục B) |

## 16. Tasks (tác vụ dài)

- **2025-11-25:** tasks ở trạng thái thử nghiệm trong lõi giao thức. Client yêu cầu server chạy nền, rồi truy vấn kết quả.
- **2026-07-28:** tasks được chuyển thành **extension** `io.modelcontextprotocol/tasks`. Thay `tasks/result` (chặn đến khi xong) bằng polling qua `tasks/get`, thêm `tasks/update` để client gửi thêm input cho tác vụ đang chạy. Bỏ `tasks/list`. Server có thể trả task handle mà không cần client yêu cầu trước. [Đã kiểm chứng]

**Khi nào dùng:** tác vụ mất hàng phút trở lên (build, ETL, render). Với tác vụ ngắn, dùng tool thông thường.

---

# PHẦN D. TRANSPORT

Transport là kênh vận chuyển JSON-RPC. Nội dung thông điệp không đổi giữa các transport.

## 17. stdio

- Host khởi chạy server dưới dạng tiến trình con.
- **stdin:** client → server. **stdout:** server → client. **stderr:** log, không được dùng cho giao thức.
- Thông điệp là JSON một dòng, phân tách bằng ký tự xuống dòng, không được chứa xuống dòng thô trong JSON.
- Tiến trình server sở hữu quyền của người dùng chạy host, nên mức tin cậy là tương đương với lệnh do người dùng tự chạy.
- Ưu: đơn giản, không cần mạng, không cần cổng. Nhược: không chia sẻ được giữa nhiều người dùng; khó scale ngang.

## 18. Streamable HTTP

Thay thế HTTP+SSE từ 2025-03-26.

### 18.1. Cơ chế

- Client gửi thông điệp bằng **HTTP POST** đến một endpoint duy nhất.
- Server trả về một trong hai:
  - `application/json`: một response đơn.
  - `text/event-stream` (SSE): luồng nhiều thông điệp (progress, notification, response trễ).
- Thông báo độc lập (không gắn request) từ 2026-07-28 đi qua `subscriptions/listen`, không dùng HTTP GET như trước. [Đã kiểm chứng]

### 18.2. Header

| Header | Vai trò |
|---|---|
| `MCP-Protocol-Version` | Phiên bản đặc tả (2025 và trước đó) |
| `Mcp-Session-Id` | Định danh phiên (chỉ đến 2025-11-25). Bị bỏ từ 2026-07-28 [Đã kiểm chứng] |
| `Mcp-Method`, `Mcp-Name` | Bắt buộc trên POST từ 2026-07-28, cho phép định tuyến ở tầng HTTP. Thêm header tùy chỉnh từ tham số tool qua `x-mcp-header` [Đã kiểm chứng] |
| `Authorization` | Bearer token theo OAuth |

### 18.3. Resumability

- Phiên bản đến 2025-11-25: dùng `Last-Event-ID` để tiếp tục luồng SSE sau khi rớt mạng.
- Từ 2026-07-28: bỏ hoàn toàn. Nếu luồng rớt, request đang chạy bị mất và client **phải gửi lại như một request mới với ID mới**. [Đã kiểm chứng]

### 18.4. Bảo mật transport

- Server phải kiểm tra header `Origin`; với `Origin` không hợp lệ, server trả **HTTP 403 Forbidden** (làm rõ ở bản 2025-11-25). Biện pháp này chống DNS rebinding khi chạy cục bộ. [Đã kiểm chứng]
- Nên bind cục bộ trên `127.0.0.1` khi chạy trên máy người dùng.

## 19. Các transport khác

- **HTTP+SSE (2024-11-05):** đã deprecated, chuyển sang Streamable HTTP. [Đã kiểm chứng]
- **Transport tùy chỉnh:** đặc tả cho phép, miễn là giữ đúng định dạng JSON-RPC và vòng đời. SDK thường cho phép cắm transport riêng.

---

# PHẦN E. ỦY QUYỀN VÀ BẢO MẬT

## 20. Ủy quyền (Authorization)

Áp dụng cho transport HTTP. stdio thường dùng thông tin xác thực trong môi trường.

### 20.1. Các vai trò OAuth

| Vai trò | Trong MCP |
|---|---|
| **Resource server** | MCP server, kiểm tra token trước khi phục vụ |
| **Authorization server** | Dịch vụ cấp token (có thể là dịch vụ riêng hoặc của nhà cung cấp) |
| **Client** | MCP client, đại diện người dùng xin token |
| **Resource owner** | Người dùng, đồng ý cấp quyền |

### 20.2. Luồng

1. Client gửi request không kèm token, server trả `401` kèm `WWW-Authenticate`.
2. Client lấy **protected resource metadata** (RFC 9728) để biết authorization server nào.
3. Client lấy metadata của authorization server (RFC 8414 hoặc OpenID Connect discovery).
4. Client đăng ký (hoặc dùng client ID đã có), rồi chuyển người dùng sang trang đồng ý.
5. Client dùng **PKCE** (RFC 7636) để đổi authorization code lấy token.
6. Client gửi token trong `Authorization: Bearer ...`.
7. Client gắn **resource indicator** (RFC 8707) để token chỉ hợp lệ với đúng server đó.

```
Client ──► MCP Server  (không token)
Client ◄── 401 + WWW-Authenticate: resource_metadata=...
Client ──► metadata tài nguyên (RFC 9728)
Client ──► metadata authorization server
Client ──► đăng ký / client ID
Client ──► trang đồng ý (người dùng)
Client ◄── authorization code
Client ──► đổi token (PKCE + resource indicator)
Client ──► MCP Server  (Authorization: Bearer ...)
```

### 20.3. Các cập nhật gần đây

- **RFC 9207 (`iss`):** authorization server nên đưa `iss` vào response, client kiểm tra trước khi đổi code. [Đã kiểm chứng]
- **Credential gắn với issuer:** client lưu credential theo issuer, không dùng lại với authorization server khác. [Đã kiểm chứng]
- **Client ID Metadata Documents:** cách đăng ký client mới ưa thích. Dynamic Client Registration (RFC 7591) bị deprecated. [Đã kiểm chứng]
- **application_type:** bắt buộc khi đăng ký động, để tránh xung đột redirect URI của OpenID Connect. [Đã kiểm chứng]

### 20.4. Quy tắc không chuyển tiếp token

Server **không được** chấp nhận token không được cấp cho chính nó (audience sai), và **không được** chuyển tiếp token nhận được sang API khác. Đây là lỗ hổng "confused deputy" và "token passthrough". [Trí nhớ]

## 21. Mô hình đe dọa

Phân thành hai nhóm: nhóm A được **đặc tả MCP chính thức** nêu tên và đưa ra biện pháp (đã kiểm chứng); nhóm B là các đe dọa phổ biến trong tài liệu nghiên cứu về LLM và agent, chưa được trang bảo mật của MCP nêu tên (nhãn `[Trí nhớ]`).

**Nhóm A. Đe dọa được đặc tả nêu tên [Đã kiểm chứng]**

| # | Đe dọa | Mô tả theo đặc tả | Biện pháp chính |
|---|---|---|---|
| A1 | **Confused deputy** (proxy server) | Proxy MCP dùng client ID tĩnh với authorization server bên thứ ba, cho phép client động. Cookie consent cũ khiến bên thứ ba bỏ qua màn hình đồng ý, mã ủy quyền bị chuyển đến kẻ tấn công | Consent theo từng client, kiểm tra exact `redirect_uri`, cookie `__Host-`, `state` dùng một lần, consent đặt trước khi chuyển hướng |
| A2 | **Token passthrough** | Server nhận token không cấp cho chính nó và chuyển tiếp xuống API khác. Phá vỡ kiểm soát truy cập, audit, và ranh giới tin cậy | Server **MUST NOT** chấp nhận token không được cấp cho mình (kiểm tra audience) |
| A3 | **SSRF** trong khám phá OAuth | Client theo URL do server độc hại cung cấp (`resource_metadata`, `authorization_servers`, các endpoint), có thể trỏ tới mạng nội bộ hoặc metadata đám mây | HTTPS bắt buộc (trừ loopback), chặn dải IP riêng, không tự theo redirect mù quáng, egress proxy, cẩn thận với TOCTOU của DNS |
| A4 | **SSRF với authorization server** | Khi hỗ trợ Client ID Metadata Documents, authorization server fetch URL do client không rõ tạo ra | Áp dụng cùng biện pháp chặn IP nội bộ |
| A5 | **State handle hijacking** (mới, bản 2026-07-28) | Server cấp handle trạng thái (ví dụ ID giỏ hàng) để giữ state không có session. Kẻ tấn công lấy hoặc đoán handle để truy cập state của người khác | Server **MUST** xác thực mọi request; handle không phải là xác thực; handle ngẫu nhiên, có hạn, gắn với user từ token |
| A6 | **Local MCP server compromise** | Server chạy cục bộ là binary thực thi được. Lệnh khởi động độc hại trong cấu hình, payload trong server, hoặc server localhost bị DNS rebinding | Consent hiển thị đầy đủ lệnh trước khi chạy (MUST), sandbox, stdio thay vì HTTP khi có thể, token hoặc IPC có quyền hạn chế nếu dùng HTTP |
| A7 | **Lỗ hổng URL ủy quyền** (XSS, RCE) | Server độc hại đưa URL `javascript:` hoặc chứa payload shell. Client mở bằng `window.open` hoặc qua shell | Chỉ cho `http`/`https` (loopback cho HTTP), từ chối `javascript:`, `data:`, `file:`, `vbscript:`, không dùng shell để mở URL, CSP |
| A8 | **stdio trong kiến trúc proxy** | XSS ở client lấy được token của proxy rồi ra lệnh spawn tiến trình qua proxy, dẫn đến RCE | Cô lập proxy, sandbox tiến trình con, ghi log, yêu cầu ủy quyền bổ sung cho lệnh nguy hiểm |
| A9 | **Mix-up attack** | Authorization server độc hại khiến client gửi mã ủy quyền của server trung thực đến token endpoint của mình | Authorization response phải có và được kiểm tra `iss` (RFC 9207). PKCE một mình **không** đủ |
| A10 | **Localhost redirect URI impersonation** | Kẻ tấn công dùng metadata URL của client hợp pháp, lắng nghe cổng localhost để nhận mã | Authorization server cảnh báo khi redirect chỉ là localhost, hiển thị hostname của redirect URI |
| A11 | **Lạm dụng CIMD** | Client ID dạng URL được chấp nhận bừa bãi | Chính sách tin cậy theo tên miền, allowlist, hiển thị hostname cho người dùng |
| A12 | **Scope quá rộng** | Token mang toàn bộ scope bị lộ, làm tăng phạm vi thiệt hại, mờ audit | Tối thiểu hóa scope, leo quyền dần qua `WWW-Authenticate` với `scope`, không quảng bá toàn bộ `scopes_supported` |

Đặc tả cũng nhắc các điểm liên quan đến session của phiên bản cũ (session hijacking) ở trang bảo mật của phiên bản 2025-11-25. [Đã kiểm chứng]

**Nhóm B. Đe dọa từ kiến thức nền [Trí nhớ, chưa đối chiếu với đặc tả MCP]**

Liệt kê theo bề mặt tấn công.

| # | Đe dọa | Mô tả | Bề mặt |
|---|---|---|---|
| T1 | **Prompt injection gián tiếp** | Nội dung từ resource, kết quả tool hoặc trang web chứa chỉ dẫn để điều khiển mô hình | Dữ liệu đầu vào mô hình |
| T2 | **Tool poisoning** | Mô tả tool chứa chỉ dẫn ẩn, lừa mô hình gọi tool khác hoặc rò rỉ dữ liệu | `description`, `annotations` |
| T3 | **Rug pull** | Server được duyệt rồi đổi mô tả hoặc hành vi tool về sau | Thay đổi `tools/list` |
| T4 | **Shadowing** | Hai server đăng ký tên tool trùng, server độc hại ghi đè | Định danh tool |
| T5 | **Confused deputy** | Server dùng quyền của mình thay mặt kẻ tấn công | Ủy quyền |
| T6 | **Token passthrough** | Server chuyển tiếp token của client sang API khác | Ủy quyền |
| T7 | **Path traversal** | Server đọc file ngoài phạm vi roots, ví dụ `../../etc/passwd` | Tool file, resource |
| T8 | **SSRF** | Server bị lừa gọi URL nội bộ | Tool fetch URL |
| T9 | **Lạm dụng sampling** | Server đốt token, đánh cắp ngữ cảnh qua sampling | Sampling |
| T10 | **Lộ dữ liệu qua tham số** | Mô hình đưa dữ liệu nhạy cảm vào tham số tool gửi đến bên thứ ba | Tham số tool |
| T11 | **Chuỗi tool độc hại** | Kết hợp nhiều tool vô hại thành hành vi nguy hiểm (ví dụ đọc file rồi gửi đi) | Tổ hợp tool |
| T12 | **Thực thi lệnh** | Tool nhận chuỗi và đưa vào shell | Tool thực thi |

**Lưu ý nghiên cứu:** T1, T2, T11 là các loại tấn công đã được mô tả trong các nghiên cứu về agent và LLM; mức độ nghiêm trọng phụ thuộc vào host có tách bạch chỉ dẫn và dữ liệu hay không. Các mục nhóm B chưa được xác nhận trong đặc tả; khi trích dẫn, cần tìm nguồn nghiên cứu gốc. [Trí nhớ]

## 22. Biện pháp phòng vệ và nguyên tắc thiết kế

### 22.1. Ở tầng host

- **Human-in-the-loop:** yêu cầu người dùng duyệt tool có tác dụng phụ. Dùng `destructiveHint` và `readOnlyHint` làm gợi ý, không làm bảo đảm.
- **Hiển thị đầy đủ:** cho người dùng thấy tên tool, tham số đầy đủ, và server nguồn.
- **Allowlist:** chỉ cho phép server và tool đã duyệt. Phát hiện thay đổi mô tả (chống T3).
- **Cách ly ngữ cảnh:** không để kết quả từ server không đáng tin trộn vào chỉ dẫn hệ thống.
- **Giới hạn sampling:** số lần, số token, và nội dung.

### 22.2. Ở tầng server

- **Least privilege:** chỉ khai báo tool cần thiết, quyền đọc/ghi tối thiểu, thư mục tối thiểu.
- **Kiểm tra đầu vào:** xác thực theo schema, chuẩn hóa đường dẫn, dùng API có tham số thay vì nối chuỗi shell.
- **Kiểm tra đầu ra:** không trả bí mật (token, khóa API) trong kết quả.
- **Idempotency:** tool có tác dụng phụ phải an toàn khi bị gọi lại.
- **Giới hạn tài nguyên:** timeout, giới hạn kích thước kết quả.

### 22.3. Nguyên tắc thiết kế chung

1. **Không tin dữ liệu bên ngoài là chỉ dẫn.** Dữ liệu chỉ là dữ liệu.
2. **Phân biệt rõ đọc và ghi.** Tool đọc và tool ghi nên tách riêng.
3. **Gắn consent với hành động cụ thể,** không gắn với cả server.
4. **Ghi log kiểm toán** cho mọi tool có tác dụng phụ.

---

# PHẦN F. VẬN HÀNH

## 23. Chi phí ngữ cảnh

Mỗi tool có `name`, `description`, `inputSchema` được đưa vào ngữ cảnh mô hình. Với nhiều server, phần này có thể lớn.

**Ước lượng:** mỗi tool thường tốn từ vài chục đến vài trăm token tùy độ dài mô tả và schema. Cần đo thực tế trên mô hình đang dùng; con số này không cố định.

**Cách giảm:**
- Mô tả ngắn, rõ, không lặp schema trong description.
- Chỉ bật server cần thiết cho tác vụ.
- Tìm kiếm tool theo nhu cầu thay vì nạp toàn bộ (tool search / lazy loading, do host hỗ trợ).
- Thứ tự tool ổn định để tăng cache prompt. [Đã kiểm chứng]
- Dùng `ttlMs` để giảm gọi `tools/list` lặp. [Đã kiểm chứng]

## 24. Quan sát, gỡ lỗi, kiểm thử

### 24.1. Công cụ

- **MCP Inspector:** giao diện web để liệt kê và gọi thử tool, resource, prompt.
  ```bash
  npx @modelcontextprotocol/inspector <lệnh-chạy-server>
  ```
- **Log:** log ra `stderr` trên stdio. Log kết nối và thời gian phản hồi ở host.
- **OpenTelemetry:** đặc tả có quy ước truyền trace context qua `_meta` (`traceparent`, `tracestate`, `baggage`). [Đã kiểm chứng]

### 24.2. Kiểm thử

| Mức | Cách kiểm |
|---|---|
| Unit | Gọi handler tool trực tiếp với tham số hợp lệ và không hợp lệ |
| Hợp đồng (contract) | Kiểm tra `inputSchema`, `outputSchema`, mã lỗi đúng với đặc tả |
| Tích hợp | Chạy server qua transport thật bằng client SDK |
| Hành vi LLM | Cho mô hình chọn tool với tập prompt cố định, đo tỷ lệ chọn đúng và tham số đúng |
| An toàn | Thử prompt injection qua dữ liệu, thử path traversal, thử token sai audience |

## 25. Triển khai và mở rộng quy mô

- **Cục bộ (stdio):** một người dùng, một tiến trình mỗi server.
- **Từ xa (Streamable HTTP):** nhiều người dùng. Cần xác thực, cô lập dữ liệu theo người dùng, và giới hạn tốc độ.
- **Stateless (2026-07-28):** dễ scale ngang vì không giữ phiên trong bộ nhớ. Trạng thái cần giữa các request phải nằm trong handle hoặc kho ngoài.
- **Cô lập:** chạy server trong container hoặc sandbox khi xử lý mã hoặc file không tin cậy.

## 26. Thiết kế tool tốt

Đây là phần quyết định mô hình có dùng tool đúng hay không.

1. **Tên động từ rõ ràng:** `search_issues`, không phải `issues` hay `do_thing`.
2. **Mô tả nói khi nào dùng và khi nào không dùng.** Mô tả là thứ mô hình đọc để chọn.
3. **Tham số ít và có kiểu rõ:** dùng `enum` cho lựa chọn cố định, `description` cho từng tham số.
4. **Đầu ra gọn:** trả đúng thông tin cần, không trả toàn bộ payload thô.
5. **Lỗi có thể hành động:** thông báo lỗi nói rõ tham số nào sai và cách sửa, để mô hình tự thử lại.
6. **Một tool làm một việc:** tránh tool "làm tất cả" với nhiều chế độ.
7. **Tách đọc và ghi.**
8. **Idempotent khi có thể.**

**Ví dụ tốt và xấu:**

| Xấu | Tốt |
|---|---|
| `run(cmd: string)` | `list_files(path: string)` với đường dẫn đã kiểm tra |
| `get_data(x)` | `get_customer_by_id(customer_id: string)` |
| Lỗi: `"failed"` | Lỗi: `"customer_id phải là chuỗi 8 chữ số; nhận được 'abc'"` |

---

# PHẦN G. THỰC HÀNH VÀ HỆ SINH THÁI

## 27. Cấu hình ở các host

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "D:\\VIBECODE"],
      "env": {}
    }
  }
}
```

**Claude Code:**

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem D:\VIBECODE
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp list
```

Trong phiên tương tác, dùng `/mcp` để xem trạng thái và xác thực.

**VS Code** (`.vscode/mcp.json`):

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

Khác biệt: Claude Desktop dùng khóa `mcpServers`, VS Code dùng `servers`.

## 28. Ví dụ server và client

### 28.1. Server Python (FastMCP)

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("ví-dụ")

@mcp.tool()
def cộng(a: int, b: int) -> int:
    """Cộng hai số nguyên và trả tổng."""
    return a + b

@mcp.resource("note://{tên}")
def đọc_ghi_chú(tên: str) -> str:
    """Đọc một ghi chú theo tên."""
    return f"Nội dung {tên}"

@mcp.prompt()
def tóm_tắt(văn_bản: str) -> str:
    """Mẫu tóm tắt."""
    return f"Hãy tóm tắt ngắn gọn:\n\n{văn_bản}"

if __name__ == "__main__":
    mcp.run()  # stdio
```

### 28.2. Client Python (tối thiểu, theo API của SDK)

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    params = StdioServerParameters(command="python", args=["server.py"])
    async with stdio_client(params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()           # phiên bản có handshake
            tools = await session.list_tools()
            print([t.name for t in tools.tools])
            result = await session.call_tool("cộng", {"a": 2, "b": 3})
            print(result.content)

asyncio.run(main())
```

Lưu ý: đoạn `session.initialize()` đúng với các phiên bản đến 2025-11-25. Với 2026-07-28, SDK dùng cơ chế stateless, nên API có thể khác. Kiểm tra phiên bản SDK bạn cài.

### 28.3. Kiểm thử bằng Inspector

```bash
npx @modelcontextprotocol/inspector python server.py
```

## 29. Hệ sinh thái

- **SDK chính thức:** TypeScript, Python, Java, Kotlin, C#, Go, Ruby, Rust, Swift, PHP. [Trí nhớ, cần kiểm tra danh sách hiện tại]
- **Server tham chiếu:** filesystem, git, fetch, memory, và các server chính thức khác.
- **Server của nhà cung cấp:** nhiều SaaS cung cấp MCP server riêng.
- **Danh mục cộng đồng:** có nhiều danh mục liệt kê server, nhưng chất lượng và độ tin cậy khác nhau. Chỉ cài server từ nguồn đáng tin và đọc mã nguồn trước khi cấp quyền.

**Tiêu chí chọn server:** nguồn gốc rõ ràng, mã nguồn công khai, phạm vi quyền tối thiểu, lịch sử cập nhật, hỗ trợ phiên bản đặc tả mình dùng.

---

# PHẦN H. PHỤ LỤC

## A. Bảng method đầy đủ theo phiên bản

| Method | Chiều | Từ phiên bản | Ghi chú |
|---|---|---|---|
| `initialize` | C→S | 2024-11-05 | Bỏ từ 2026-07-28 |
| `notifications/initialized` | C→S | 2024-11-05 | Bỏ từ 2026-07-28 |
| `ping` | Hai chiều | 2024-11-05 | Bỏ từ 2026-07-28 |
| `server/discover` | C→S | 2026-07-28 | Bắt buộc với server |
| `tools/list` | C→S | 2024-11-05 | |
| `tools/call` | C→S | 2024-11-05 | |
| `resources/list` | C→S | 2024-11-05 | |
| `resources/read` | C→S | 2024-11-05 | |
| `resources/templates/list` | C→S | 2024-11-05 | |
| `resources/subscribe` | C→S | 2024-11-05 | Thay bằng `subscriptions/listen` từ 2026-07-28 |
| `resources/unsubscribe` | C→S | 2024-11-05 | Tương tự |
| `prompts/list` | C→S | 2024-11-05 | |
| `prompts/get` | C→S | 2024-11-05 | |
| `subscriptions/listen` | C→S | 2026-07-28 | Luồng thông báo dài |
| `completion/complete` | C→S | 2024-11-05 | |
| `logging/setLevel` | C→S | 2024-11-05 | Bỏ từ 2026-07-28 |
| `sampling/createMessage` | S→C | 2024-11-05 | Deprecated 2026-07-28 |
| `roots/list` | S→C | 2024-11-05 | Deprecated 2026-07-28 |
| `elicitation/create` | S→C | 2025-06-18 | |
| `tasks/get`, `tasks/update` | C→S | 2026-07-28 | Thuộc extension tasks |
| `notifications/tools/list_changed` | S→C | 2024-11-05 | |
| `notifications/resources/list_changed` | S→C | 2024-11-05 | |
| `notifications/resources/updated` | S→C | 2024-11-05 | |
| `notifications/prompts/list_changed` | S→C | 2024-11-05 | |
| `notifications/progress` | S→C | 2024-11-05 | Gắn với request |
| `notifications/cancelled` | Hai chiều | 2024-11-05 | |
| `notifications/message` | S→C | 2024-11-05 | Deprecated (logging) |
| `notifications/roots/list_changed` | C→S | 2024-11-05 | Bỏ từ 2026-07-28 |

Ghi chú: bảng này tổng hợp từ changelog và kiến thức nền. Danh sách đầy đủ nên đối chiếu với schema chính thức của đúng phiên bản.

## B. Bảng mã lỗi

| Mã | Tên | Ghi chú |
|---|---|---|
| -32700 | Parse error | JSON-RPC chuẩn |
| -32600 | Invalid Request | JSON-RPC chuẩn |
| -32601 | Method not found | JSON-RPC chuẩn |
| -32602 | Invalid params | Dùng cho resource không tồn tại từ 2026-07-28 |
| -32603 | Internal error | JSON-RPC chuẩn |
| -32002 | Resource not found | Phiên bản cũ, đã đổi sang -32602 |
| -32000 đến -32019 | Dành cho triển khai | Đã được grandfathered |
| -32020 đến -32099 | Dành cho đặc tả | Từ 2026-07-28 [Đã kiểm chứng] |
| -32020 | HeaderMismatch | Trước đây là -32001 [Đã kiểm chứng] |
| -32021 | MissingRequiredClientCapability | Trước đây là -32003 [Đã kiểm chứng] |
| -32022 | UnsupportedProtocolVersion | Trước đây là -32004 [Đã kiểm chứng] |

## C. Glossary song ngữ Anh–Việt

| Thuật ngữ (Anh) | Tiếng Việt | Giải thích ngắn |
|---|---|---|
| Authorization server | Máy chủ cấp quyền | Phát hành token |
| Capability | Khả năng | Thứ một bên hỗ trợ, khai báo khi đàm phán |
| Client | Máy khách | Kết nối 1:1 đến một server |
| Consent | Đồng ý | Người dùng duyệt hành động |
| Elicitation | Hỏi người dùng | Server yêu cầu thông tin |
| Host | Máy chủ ứng dụng / ứng dụng chủ | Ứng dụng AI chứa mô hình và client |
| Human-in-the-loop | Người tham gia vòng lặp | Người dùng duyệt trước khi thực thi |
| Idempotent | Lũy đẳng | Gọi lại nhiều lần cho kết quả như một lần |
| InputRequiredResult | Kết quả cần thêm đầu vào | Thành phần của MRTR |
| Least privilege | Quyền tối thiểu | Chỉ cấp quyền cần thiết |
| MRTR | Nhiều vòng trao đổi | Multi Round-Trip Requests |
| Notification | Thông báo | Thông điệp không cần phản hồi |
| Primitive | Thành phần cơ bản | Tools, resources, prompts, sampling, elicitation, roots |
| Prompt injection | Tiêm chỉ dẫn | Nhúng lệnh vào dữ liệu để điều khiển mô hình |
| Protocol version | Phiên bản đặc tả | Định danh theo ngày |
| Resource | Tài nguyên | Dữ liệu chỉ đọc theo URI |
| Resource indicator | Chỉ định tài nguyên | RFC 8707, khóa token với audience |
| Root | Gốc phạm vi | Ranh giới thư mục gợi ý |
| Sampling | Lấy mẫu sinh nội dung | Server nhờ mô hình của host |
| Server | Máy chủ cung cấp | Cung cấp tools, resources, prompts |
| Stateless | Không trạng thái | Mỗi request tự đủ thông tin |
| Tasks | Tác vụ dài | Chạy nền và truy vấn kết quả |
| Tool | Công cụ | Hàm mô hình gọi để hành động |
| Tool poisoning | Đầu độc công cụ | Sửa mô tả để lừa mô hình |
| Transport | Lớp vận chuyển | stdio, Streamable HTTP |

## D. Danh mục nguồn tham khảo

**D.1. Đặc tả MCP chính thức (đã tra cứu, phiên bản mới nhất là 2026-07-28)**

| Nguồn | URL | Ghi chú |
|---|---|---|
| Trang chỉ mục tài liệu (llms.txt) | https://modelcontextprotocol.io/llms.txt | Danh sách toàn bộ trang, theo phiên bản |
| Đặc tả bản mới nhất | https://modelcontextprotocol.io/specification/2026-07-28/index.md | Trang gốc của đặc tả |
| Đặc tả bản nháp | https://modelcontextprotocol.io/specification/draft/ | Chưa phát hành, có thể thay đổi |
| Changelog 2026-07-28 | https://modelcontextprotocol.io/specification/latest/changelog | Liệt kê thay đổi lớn, nhỏ, deprecated |
| Danh sách deprecated | https://modelcontextprotocol.io/specification/2026-07-28/deprecated | Registry các tính năng đang deprecated |
| Versioning và tương thích | https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning | Mô hình hai thời đại (modern/legacy) |
| Lifecycle (phiên bản cũ) | https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle | Handshake `initialize` |
| Schema TypeScript | https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.ts | Nguồn chuẩn để đối chiếu kiểu dữ liệu |
| Schema JSON | https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.json | Dùng để validate |
| Kiến trúc | https://modelcontextprotocol.io/specification/2026-07-28/architecture/index | Host, client, server |
| Transport stdio | https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio | |
| Transport Streamable HTTP | https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http | |
| Ủy quyền | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/index | |
| Khám phá authorization server | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/authorization-server-discovery | |
| Đăng ký client | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration | |
| Bảo mật ủy quyền | https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations | |
| MRTR | https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/mrtr | Mục 13 của tài liệu này |
| Subscriptions | https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/subscriptions | |
| Cancellation | https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/cancellation | |
| Progress | https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/progress | |
| Tools (phía server) | https://modelcontextprotocol.io/specification/2026-07-28/server/tools | |
| Resources | https://modelcontextprotocol.io/specification/2026-07-28/server/resources | |
| Prompts | https://modelcontextprotocol.io/specification/2026-07-28/server/prompts | |
| Discover | https://modelcontextprotocol.io/specification/2026-07-28/server/discover | |
| Caching | https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/caching | |
| Completion | https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/completion | |
| Logging | https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/logging | |
| Pagination | https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/pagination | |
| Roots (client) | https://modelcontextprotocol.io/specification/2026-07-28/client/roots | Deprecated |
| Sampling (client) | https://modelcontextprotocol.io/specification/2026-07-28/client/sampling | Deprecated |
| Elicitation (client) | https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation | |

**D.2. Tài liệu hướng dẫn (docs)**

- Giới thiệu: https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro
- Kiến trúc: https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture
- Khái niệm server: https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts
- Khái niệm client: https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts
- Versioning: https://modelcontextprotocol.io/docs/2026-07-28/learn/versioning
- Kết nối server cục bộ: https://modelcontextprotocol.io/docs/2026-07-28/develop/connect-local-servers
- Kết nối server từ xa: https://modelcontextprotocol.io/docs/2026-07-28/develop/connect-remote-servers
- Xây server: https://modelcontextprotocol.io/docs/2026-07-28/develop/build-server
- Xây client: https://modelcontextprotocol.io/docs/2026-07-28/develop/build-client
- Thực hành tốt cho client: https://modelcontextprotocol.io/docs/2026-07-28/develop/clients/client-best-practices
- Danh sách SDK: https://modelcontextprotocol.io/docs/2026-07-28/sdk
- Ví dụ server: https://modelcontextprotocol.io/examples
- Bảo mật, ủy quyền: https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/authorization, https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices, https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/local-server-security
- Inspector: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector
- Gỡ lỗi: https://modelcontextprotocol.io/docs/2026-07-28/tools/debugging

**D.3. Phần mở rộng (extensions)**

- Tổng quan: https://modelcontextprotocol.io/extensions/overview
- Ma trận client: https://modelcontextprotocol.io/extensions/client-matrix
- Tasks: https://modelcontextprotocol.io/extensions/tasks/overview
- MCP Apps: https://modelcontextprotocol.io/extensions/apps/overview
- Skills: https://modelcontextprotocol.io/extensions/skills/overview
- Ủy quyền doanh nghiệp: https://modelcontextprotocol.io/extensions/auth/enterprise-managed-authorization

**D.4. SEP (đề xuất cải tiến đặc tả)**

Chỉ mục: https://modelcontextprotocol.io/seps/index.md. Các SEP được dùng trong tài liệu này:

| SEP | Chủ đề | Liên quan mục |
|---|---|---|
| SEP-2575 | Stateless MCP | 9.2, 18.3 |
| SEP-2567 | Sessionless MCP | 2, 9.2 |
| SEP-2322 | MRTR | 13.2 |
| SEP-2663 | Tasks extension | 16 |
| SEP-2577 | Deprecate Roots, Sampling, Logging | 12, 14.3 |
| SEP-2596 | Vòng đời và deprecation | 2 |
| SEP-2549 | TTL cho kết quả list | 14.5 |
| SEP-2243 | Chuẩn hóa HTTP header | 18.2 |
| SEP-2468 | Tham số `iss` (RFC 9207) | 20.3 |
| SEP-2352 | Credential gắn với issuer | 20.3 |
| SEP-2164 | Lỗi resource not found | 11.2 |
| SEP-1036 | URL mode elicitation | 12.2 |
| SEP-1577 | Sampling có tool | 12.1 |
| SEP-1686 | Tasks (bản đầu) | 16 |
| SEP-2106 | JSON Schema 2020-12 | 11.1 |
| SEP-1303 | Lỗi validate đầu vào là lỗi thực thi tool | 15 |
| SEP-986 | Định dạng tên tool | 11.1 |
| SEP-1024 | Yêu cầu bảo mật client cho server cục bộ | 22 |
| SEP-1730 | Hệ thống phân tầng SDK | 29 |
| SEP-1850 | Quy trình SEP qua PR | D.4 |

Danh sách SEP đầy đủ: https://modelcontextprotocol.io/seps/index.md

**D.5. Kho mã nguồn (repository)**

Repository chính: https://github.com/modelcontextprotocol/modelcontextprotocol. Các thư mục cấp cao (đã kiểm tra):

| Thư mục | Nội dung |
|---|---|
| `schema/` | Schema chuẩn: `schema/2026-07-28/schema.ts` và `schema.json` (và các phiên bản khác) |
| `docs/` | Nguồn của trang tài liệu |
| `seps/` | Nguồn các SEP |
| `blog/` | Bài blog |
| `plugins/mcp-spec/` | Plugin hỗ trợ đặc tả |
| `scripts/` | Script build và kiểm tra |
| `tools/sep-automation/` | Tự động hóa quy trình SEP |
| `.github/` | Cấu hình CI và template |

Tệp cấp cao quan trọng: `README.md`, `CONTRIBUTING.md`, `GOVERNANCE.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `ANTITRUST.md`, `AI_POLICY.md`, `AGENTS.md`, `CLAUDE.md`, `MAINTAINERS.md`.

Đặc tả nguồn (các file markdown của đặc tả) nằm trong repository và được dựng thành trang web. Đường dẫn chính xác trong repository chưa được xác nhận trong lần tra cứu này; nên xem trực tiếp trên GitHub hoặc tệp `llms.txt`.

**D.6. Chuẩn liên quan (IETF, W3C, JSON-RPC)**

- JSON-RPC 2.0 Specification: https://www.jsonrpc.org/specification
- RFC 2119 (MUST, SHOULD, MAY): https://www.rfc-editor.org/rfc/rfc2119
- OAuth 2.0 (RFC 6749): https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.1 (draft): https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/
- RFC 7591 (Dynamic Client Registration, deprecated trong MCP): https://www.rfc-editor.org/rfc/rfc7591
- RFC 7636 (PKCE): https://www.rfc-editor.org/rfc/rfc7636
- RFC 8414 (Authorization Server Metadata): https://www.rfc-editor.org/rfc/rfc8414
- RFC 8707 (Resource Indicators): https://www.rfc-editor.org/rfc/rfc8707
- RFC 9207 (Issuer Identification): https://www.rfc-editor.org/rfc/rfc9207
- RFC 9728 (Protected Resource Metadata): https://www.rfc-editor.org/rfc/rfc9728
- JSON Schema 2020-12: https://json-schema.org/specification
- LSP: https://microsoft.github.io/language-server-protocol/

**D.7. Nghiên cứu về an toàn (chưa tra cứu, cần bổ sung)**

- Greshake và cộng sự (2023), "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection". Tìm bản gốc trên arXiv trước khi trích dẫn. [Trí nhớ]
- Các phân tích về tool poisoning và rug pull trong hệ sinh thái MCP: chưa có nguồn đã kiểm chứng trong tài liệu này. [Chưa xác nhận]

**Ghi chú về kiểm chứng (lần tra cứu này):**
- Đã xác nhận bằng trang chính thức: phiên bản mới nhất là 2026-07-28; mô hình không có handshake cho bản hiện đại; mã lỗi `-32022` cho phiên bản không hỗ trợ; server hiện đại phải hỗ trợ `server/discover`; SEP và đường dẫn tài liệu như trên.
- Chưa xác nhận: tag phát hành mới nhất trên GitHub (trang không hiển thị), danh sách SDK chính thức đầy đủ (cần xem trang `/sdk`), đường dẫn đặc tả nguồn trong repository.

## E. Câu hỏi mở và điểm cần kiểm chứng

1. **Phiên bản chuẩn cho dự án của bạn:** nên chọn 2025-11-25 (ổn định, nhiều SDK hỗ trợ) hay 2026-07-28 (mới, stateless). Cần kiểm tra SDK nào hỗ trợ phiên bản nào.
2. **Danh sách SDK chính thức:** cần đối chiếu với README của repository.
3. **Chi phí token thực tế:** cần đo trên mô hình cụ thể.
4. **Tính hiệu quả của annotation** trong việc ngăn tool có tác dụng phụ: cần thử nghiệm.
5. **Mức độ hỗ trợ của từng host** với các tính năng mới (MRTR, subscriptions/listen, tasks extension): khác nhau theo host và phiên bản.
6. **Các mục đánh dấu [Trí nhớ]** trong tài liệu này: nên đối chiếu với đặc tả gốc trước khi trích dẫn chính thức.

---

**Tóm lược để đọc nhanh:**
- MCP = giao thức JSON-RPC giữa host (có client) và server, gồm tools, resources, prompts và các primitive phía client.
- Phiên bản 2026-07-28 bỏ handshake, chuyển sang stateless, dùng MRTR và `subscriptions/listen`, deprecated roots, sampling, logging.
- An toàn dựa vào consent của người dùng, quyền tối thiểu, và phòng vệ chống prompt injection, tool poisoning, token passthrough.
