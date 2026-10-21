# Tài liệu tổng quan về MCP (Model Context Protocol)

> Viết cho: người mới tìm hiểu MCP, đọc tiếng Việt, muốn hiểu kiến trúc, thuật ngữ và luồng kết nối.

---

## Mục lục

1. [MCP là gì?](#1-mcp-là-gì)
2. [Vì sao cần MCP? (Vấn đề MCP giải quyết)](#2-vì-sao-cần-mcp-vấn-đề-mcp-giải-quyết)
3. [Kiến trúc tổng thể](#3-kiến-trúc-tổng-thể)
4. [Các vai trò: Host, Client, Server](#4-các-vai-trò-host-client-server)
5. [Các thành phần (Primitives)](#5-các-thành-phần-primitives)
6. [Giao thức nền: JSON-RPC 2.0](#6-giao-thức-nền-json-rpc-20)
7. [Transport: cách client và server nói chuyện](#7-transport-cách-client-và-server-nói-chuyện)
8. [Vòng đời kết nối (Lifecycle)](#8-vòng-đời-kết-nối-lifecycle)
9. [Luồng kết nối chi tiết (Flow)](#9-luồng-kết-nối-chi-tiết-flow)
10. [Xác thực và bảo mật](#10-xác-thực-và-bảo-mật)
11. [Cấu hình MCP trong thực tế](#11-cấu-hình-mcp-trong-thực-tế)
12. [Tạo một MCP server đơn giản](#12-tạo-một-mcp-server-đơn-giản)
13. [Bảng thuật ngữ (Glossary)](#13-bảng-thuật-ngữ-glossary)
14. [Tóm tắt](#14-tóm-tắt)

---

## 1. MCP là gì?

**MCP (Model Context Protocol)** là một **giao thức mở** (open standard) cho phép ứng dụng AI (LLM) kết nối với **dữ liệu bên ngoài và công cụ bên ngoài** theo một cách **chuẩn hóa**.

Có thể hình dung MCP như **cổng USB-C cho AI**: thay vì mỗi ứng dụng AI tự viết một cách kết nối riêng cho từng dịch vụ (GitHub, Google Drive, database, Slack...), tất cả đều nói chung một ngôn ngữ MCP. Viết connector một lần, mọi AI client hỗ trợ MCP đều dùng được.

- **Người phát triển ban đầu:** Anthropic (công bố cuối 2024), nay là dự án mã nguồn mở có đặc tả công khai.
- **Nền tảng kỹ thuật:** JSON-RPC 2.0.
- **SDK chính thức:** TypeScript, Python, Java, Kotlin, C#, Go và nhiều ngôn ngữ khác.

---

## 2. Vì sao cần MCP? (Vấn đề MCP giải quyết)

Trước MCP, để một LLM dùng được dữ liệu/công cụ bên ngoài, phải làm:

```
  N ứng dụng AI  ×  M dịch vụ  =  N × M  tích hợp riêng lẻ
```

Mỗi tích hợp có cách xác thực, định dạng, cách mô tả công cụ khác nhau. Khi số lượng tăng, việc bảo trì trở nên bất khả thi.

Với MCP:

```
  N ứng dụng AI  +  M dịch vụ  =  N + M  (mỗi bên implement MCP một lần)
```

| Trước MCP | Với MCP |
|---|---|
| Mỗi app AI tự viết plugin/connector riêng | Viết MCP server một lần, dùng ở mọi MCP client |
| Định dạng mô tả công cụ khác nhau | Thống nhất: tools / resources / prompts |
| Khó kiểm soát quyền truy cập | Có khái niệm host, consent, phân quyền rõ ràng |

---

## 3. Kiến trúc tổng thể

```
┌───────────────────────────────────────────────────────────────┐
│                     HOST (ứng dụng AI)                        │
│        Ví dụ: Claude Desktop, Claude Code, VS Code, Cursor    │
│                                                               │
│   ┌────────────┐   ┌────────────┐   ┌────────────┐            │
│   │ LLM        │   │ Quản lý    │   │ Giao diện  │            │
│   │ (mô hình)  │◄─►│ quyền/     │◄─►│ người dùng │            │
│   │            │   │ consent    │   │            │            │
│   └────────────┘   └────────────┘   └────────────┘            │
│         ▲                                                     │
│         │ (mỗi server = 1 client riêng)                       │
│   ┌─────┴──────┐        ┌────────────┐        ┌────────────┐  │
│   │ MCP Client │        │ MCP Client │        │ MCP Client │  │
│   │     #1     │        │     #2     │        │     #3     │  │
│   └─────┬──────┘        └─────┬──────┘        └─────┬──────┘  │
└─────────┼─────────────────────┼─────────────────────┼─────────┘
          │  JSON-RPC 2.0       │                     │
          │  (qua transport)    │                     │
    ┌─────▼──────┐        ┌─────▼──────┐        ┌─────▼──────┐
    │ MCP Server │        │ MCP Server │        │ MCP Server │
    │ (GitHub)   │        │ (Database) │        │ (Files)    │
    └─────┬──────┘        └─────┬──────┘        └─────┬──────┘
          ▼                     ▼                     ▼
     GitHub API            PostgreSQL           Hệ thống file
```

**Ý chính:** Một host có thể chứa **nhiều client**, mỗi client kết nối **đúng một server** (quan hệ 1:1). Server không biết về nhau; host là nơi điều phối tất cả.

---

## 4. Các vai trò: Host, Client, Server

### 4.1. Host

- Là **ứng dụng mà người dùng thực sự dùng** (chatbot, IDE, CLI...).
- **Chứa LLM** và quyết định khi nào gọi công cụ.
- **Tạo và quản lý** các MCP client.
- **Chịu trách nhiệm bảo mật:** hỏi consent (đồng ý) của người dùng trước khi chạy công cụ, kiểm soát quyền truy cập dữ liệu, lọc đầu ra.
- Ví dụ: Claude Desktop, Claude Code, VS Code (Copilot Agent mode), Cursor, Windsurf, các ứng dụng tự xây bằng Agent SDK.

### 4.2. Client (MCP Client)

- Là **thành phần nằm trong host**, làm nhiệm vụ **kết nối 1:1 với một server**.
- Chịu trách nhiệm:
  - Thực hiện **handshake** (khởi tạo kết nối, đàm phán khả năng).
  - Gửi **request** (ví dụ `tools/list`, `tools/call`) và nhận **response**.
  - Nhận **notification** (thông báo) từ server.
  - Quản lý vòng đời kết nối (kết nối, timeout, ngắt, kết nối lại).
- Client **không phải** là ứng dụng người dùng nhìn thấy; nó là lớp kỹ thuật bên trong host.

### 4.3. Server (MCP Server)

- Là **chương trình cung cấp khả năng** cho AI: truy cập file, gọi API, truy vấn database...
- Có thể chạy **cục bộ** (subprocess trên máy người dùng) hoặc **từ xa** (dịch vụ trên mạng).
- Khai báo **capabilities** (khả năng nó hỗ trợ) và **danh sách** tools / resources / prompts.
- Thực thi công cụ khi nhận lệnh gọi, rồi trả kết quả.
- Server **không thấy toàn bộ cuộc hội thoại** và không biết về các server khác; nó chỉ nhận đúng request mà client gửi.

### 4.4. Bảng so sánh nhanh

| Vai trò | Là gì | Ví dụ | Ai kiểm soát |
|---|---|---|---|
| **Host** | Ứng dụng AI người dùng dùng | Claude Desktop, VS Code | Người dùng + ứng dụng |
| **Client** | Bộ kết nối 1:1 nằm trong host | (lớp ẩn bên trong) | Host |
| **Server** | Cung cấp công cụ/dữ liệu | MCP GitHub, MCP Postgres | Nhà phát triển server |

---

## 5. Các thành phần (Primitives)

"Primitives" là **các loại khả năng cơ bản** mà MCP định nghĩa. Chia thành hai nhóm: **phía server cung cấp** và **phía client cung cấp**.

### 5.1. Phía server cung cấp

#### a) Tools (Công cụ)

- Là **hàm mà LLM có thể gọi** để thực hiện hành động hoặc truy vấn: `search_issues`, `create_file`, `run_query`...
- **Được điều khiển bởi model**: LLM quyết định khi nào gọi (tuy host thường yêu cầu người dùng xác nhận).
- Có thể **có tác dụng phụ** (ghi file, gửi email...), nên cần cẩn trọng.
- Định nghĩa gồm: `name`, `title`, `description`, `inputSchema` (JSON Schema mô tả tham số), `outputSchema` (tùy chọn), `annotations` (gợi ý như `readOnlyHint`, `destructiveHint`).

Ví dụ định nghĩa một tool:

```json
{
  "name": "get_weather",
  "description": "Lấy thời tiết hiện tại của một thành phố",
  "inputSchema": {
    "type": "object",
    "properties": {
      "city": { "type": "string", "description": "Tên thành phố" }
    },
    "required": ["city"]
  }
}
```

#### b) Resources (Tài nguyên)

- Là **dữ liệu chỉ đọc** được xác định bằng **URI**: nội dung file, bản ghi database, trang tài liệu, log...
- **Được điều khiển bởi ứng dụng (host)**: host quyết định nội dung nào được đưa vào ngữ cảnh.
- Ví dụ URI: `file:///home/user/notes.md`, `postgres://db/customers/42`.
- Có thể có **resource templates** (URI có tham số, ví dụ `weather://{city}`).
- Có thể **subscribe**: client đăng ký để được báo khi resource thay đổi.

#### c) Prompts (Mẫu lệnh)

- Là **template prompt có sẵn** do server cung cấp, có tham số, để người dùng chọn và dùng.
- **Được điều khiển bởi người dùng**: thường hiển thị như lệnh slash (ví dụ `/summarize-pr`) hoặc menu.
- Giúp chuẩn hóa cách hỏi cho một tác vụ lặp lại.

### 5.2. Phía client cung cấp

Server có thể **yêu cầu ngược lại** client (hoặc host) một số khả năng:

#### d) Sampling

- Server **nhờ LLM của host sinh văn bản** (completion) thay vì tự gọi API model.
- Host vẫn kiểm soát model, chi phí và quyền; server không cần API key riêng cho LLM.
- Ví dụ: một server phân tích log muốn LLM tóm tắt một đoạn log.

#### e) Elicitation

- Server **hỏi thêm thông tin từ người dùng** trong lúc thực thi (ví dụ: "Bạn muốn xác nhận xóa không?", "Nhập mật khẩu staging").
- Client hiển thị form/hộp thoại và trả dữ liệu về server.

#### f) Roots

- Client **cho server biết phạm vi được phép làm việc**: ví dụ danh sách thư mục dự án `file:///home/user/project`.
- Đây là ranh giới để server biết nên tìm/đọc ở đâu, không phải là cơ chế bảo mật tuyệt đối.

### 5.3. Tiện ích và thông báo (Utilities)

| Thành phần | Tác dụng |
|---|---|
| **Notifications** | Thông báo một chiều, không cần trả lời. Ví dụ `notifications/tools/list_changed` báo danh sách tool đã thay đổi. |
| **Progress** | Báo tiến độ tác vụ dài. |
| **Cancellation** | Huỷ một request đang chạy. |
| **Logging** | Server gửi log về client để debug. |
| **Pagination** | Chia nhỏ danh sách lớn thành nhiều trang. |
| **Ping** | Kiểm tra kết nối còn sống. |
| **Completion** | Gợi ý giá trị cho tham số (autocomplete). |

### 5.4. Bảng tổng hợp primitives

| Primitive | Thuộc về | Ai điều khiển | Bản chất |
|---|---|---|---|
| Tools | Server | Model (LLM) | Hành động / hàm thực thi |
| Resources | Server | Host/ứng dụng | Dữ liệu chỉ đọc theo URI |
| Prompts | Server | Người dùng | Template lệnh có sẵn |
| Sampling | Client | Server yêu cầu, host duyệt | Nhờ LLM của host sinh nội dung |
| Elicitation | Client | Server yêu cầu, người dùng trả lời | Hỏi thêm thông tin người dùng |
| Roots | Client | Client khai báo | Phạm vi thư mục/URI được phép |

---

## 6. Giao thức nền: JSON-RPC 2.0

MCP trao đổi tin nhắn theo **JSON-RPC 2.0**. Có ba loại tin nhắn:

| Loại | Đặc điểm | Ví dụ |
|---|---|---|
| **Request** | Có `id`, cần phản hồi | Client gọi `tools/call` |
| **Response** | Cùng `id` với request; có `result` hoặc `error` | Server trả kết quả |
| **Notification** | Không có `id`, không cần phản hồi | `notifications/initialized` |

**Ví dụ gọi một tool:**

Request (client → server):
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": { "city": "Hà Nội" }
  }
}
```

Response (server → client):
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      { "type": "text", "text": "Hà Nội: 28°C, có mây" }
    ],
    "isError": false
  }
}
```

### Các method (phương thức) thường gặp

| Method | Chiều | Mục đích |
|---|---|---|
| `initialize` | Client → Server | Bắt đầu phiên, đàm phán khả năng |
| `notifications/initialized` | Client → Server | Báo đã khởi tạo xong |
| `ping` | Hai chiều | Kiểm tra kết nối |
| `tools/list` | Client → Server | Lấy danh sách tool |
| `tools/call` | Client → Server | Gọi một tool |
| `resources/list` | Client → Server | Lấy danh sách resource |
| `resources/read` | Client → Server | Đọc nội dung resource |
| `resources/subscribe` | Client → Server | Theo dõi thay đổi resource |
| `prompts/list` | Client → Server | Lấy danh sách prompt |
| `prompts/get` | Client → Server | Lấy nội dung prompt đã điền tham số |
| `sampling/createMessage` | Server → Client | Nhờ LLM sinh nội dung |
| `elicitation/create` | Server → Client | Hỏi thông tin người dùng |
| `roots/list` | Server → Client | Hỏi danh sách roots |
| `notifications/tools/list_changed` | Server → Client | Báo tool thay đổi |

> Lưu ý: MCP là **song hướng** — không chỉ client gọi server; server cũng có thể gọi ngược client (sampling, elicitation, roots).

---

## 7. Transport: cách client và server nói chuyện

**Transport** là **kênh vận chuyển** các tin nhắn JSON-RPC. Cùng một nội dung JSON-RPC, có thể đi qua các kênh khác nhau.

### 7.1. stdio (Standard Input/Output)

- Host **khởi chạy server như một tiến trình con** (subprocess).
- Tin nhắn đi qua **stdin/stdout**; log/debug đi qua **stderr**.
- Dùng cho server **chạy cục bộ** trên máy người dùng.
- Ưu điểm: đơn giản, không cần mạng, không cần mở cổng.
- Ví dụ host chạy: `npx -y @modelcontextprotocol/server-filesystem /đường/dẫn`

### 7.2. Streamable HTTP

- Dùng cho server **từ xa**, qua HTTP.
- Client gửi request bằng HTTP POST; server có thể trả JSON trực tiếp hoặc mở luồng SSE (Server-Sent Events) để stream nhiều tin nhắn.
- Có thể dùng **session ID** để duy trì phiên.
- Đây là transport từ xa hiện đại, thay thế cho cơ chế **HTTP+SSE** cũ (đã bị deprecated trong đặc tả).

### 7.3. So sánh

| Tiêu chí | stdio | Streamable HTTP |
|---|---|---|
| Vị trí server | Cùng máy, subprocess | Máy khác / đám mây |
| Vòng đời | Host quản lý tiến trình | Server độc lập, chạy liên tục |
| Xác thực | Thường dựa vào quyền hệ điều hành / biến môi trường | OAuth 2.1 / token |
| Ví dụ | Filesystem server, Git server cục bộ | Dịch vụ SaaS có MCP (Notion, Linear...) |

---

## 8. Vòng đời kết nối (Lifecycle)

Một phiên MCP đi qua 3 giai đoạn:

```
 ┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
 │ 1. Khởi tạo  │ ──► │ 2. Hoạt động     │ ──► │ 3. Tắt máy   │
 │ (initialize) │     │ (requests/notif) │     │ (shutdown)   │
 └──────────────┘     └──────────────────┘     └──────────────┘
```

1. **Khởi tạo (Initialization):**
   - Client gửi `initialize` với `protocolVersion`, `capabilities` (khả năng phía client), `clientInfo`.
   - Server trả về `protocolVersion` (phiên bản nó chọn), `capabilities` (tools/resources/prompts, logging...), `serverInfo`.
   - Client gửi `notifications/initialized`.
   - Nếu phiên bản không tương thích, hai bên ngắt kết nối.
2. **Hoạt động (Operation):**
   - Trao đổi request/response/notification theo khả năng đã thống nhất.
   - Chỉ dùng những capability mà cả hai bên đã khai báo.
3. **Tắt máy (Shutdown):**
   - Với stdio: đóng stdin, chờ tiến trình thoát; nếu cần thì kết thúc tiến trình.
   - Với HTTP: đóng kết nối / kết thúc session.

**Ví dụ handshake:**

```json
// Client → Server
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{
  "protocolVersion":"2025-06-18",
  "capabilities":{"sampling":{},"roots":{"listChanged":true}},
  "clientInfo":{"name":"ví-dụ-client","version":"1.0.0"}}}

// Server → Client
{"jsonrpc":"2.0","id":1,"result":{
  "protocolVersion":"2025-06-18",
  "capabilities":{"tools":{"listChanged":true},"resources":{}},
  "serverInfo":{"name":"ví-dụ-server","version":"1.0.0"}}}

// Client → Server
{"jsonrpc":"2.0","method":"notifications/initialized"}
```

> Các phiên bản đặc tả (protocolVersion) có dạng ngày, ví dụ `2024-11-05`, `2025-03-26`, `2025-06-18`. Phiên bản mới nhất xem tại trang đặc tả chính thức.

---

## 9. Luồng kết nối chi tiết (Flow)

### 9.1. Luồng tổng quát: từ lúc khởi động đến khi có câu trả lời

```
 Người dùng        HOST              MCP Client        MCP Server        LLM / API
    │               │                     │                 │                │
    │ 1. Mở app     │                     │                 │                │
    │──────────────►│                     │                 │                │
    │               │ 2. Đọc config       │                 │                │
    │               │ 3. Tạo client       │                 │                │
    │               │────────────────────►│ 4. Khởi chạy    │                │
    │               │                     │  server (stdio) │                │
    │               │                     │────────────────►│                │
    │               │                     │ 5. initialize   │                │
    │               │                     │────────────────►│                │
    │               │                     │ 6. capabilities │                │
    │               │                     │◄────────────────│                │
    │               │                     │ 7. initialized  │                │
    │               │                     │────────────────►│                │
    │               │                     │ 8. tools/list   │                │
    │               │                     │────────────────►│                │
    │               │                     │ 9. danh sách    │                │
    │               │                     │◄────────────────│                │
    │               │ 10. Đưa mô tả tool  │                 │                │
    │               │     vào ngữ cảnh   │                 │                │
    │               │─────────────────────────────────────────────────────────►│
    │ 11. Hỏi:      │                     │                 │                │
    │  "Thời tiết?" │                     │                 │                │
    │──────────────►│────────────────────────────────────────────────────────►│
    │               │                     │                 │ 12. LLM quyết  │
    │               │                     │                 │  định gọi tool │
    │               │◄────────────────────────────────────────────────────────│
    │ 13. Xin xác   │                     │                 │                │
    │  nhận (consent)│                    │                 │                │
    │──────────────►│                     │                 │                │
    │               │ 14. tools/call      │                 │                │
    │               │────────────────────►│────────────────►│                │
    │               │                     │ 15. server gọi  │                │
    │               │                     │     API thật    │                │
    │               │                     │ 16. result      │                │
    │               │                     │◄────────────────│                │
    │               │ 17. Đưa kết quả     │                 │                │
    │               │     cho LLM         │                 │                │
    │               │─────────────────────────────────────────────────────────►│
    │               │◄─────────────────────────────────────────────────────────│
    │ 18. Câu trả   │                     │                 │                │
    │     lời cuối  │                     │                 │                │
    │◄──────────────│                     │                 │                │
```

### 9.2. Giải thích từng bước

| Bước | Ai làm | Nội dung |
|---|---|---|
| 1 | Người dùng | Mở ứng dụng AI (host) |
| 2 | Host | Đọc file cấu hình (danh sách server cần kết nối) |
| 3 | Host | Tạo một MCP client cho mỗi server |
| 4 | Client | Khởi chạy server (stdio) hoặc mở kết nối HTTP |
| 5–7 | Client ↔ Server | Handshake: `initialize` → `initialized` |
| 8–9 | Client ↔ Server | Khám phá khả năng: `tools/list`, `resources/list`, `prompts/list` |
| 10 | Host | Đưa tên, mô tả, schema các tool vào ngữ cảnh của LLM |
| 11 | Người dùng | Đặt câu hỏi |
| 12 | LLM | Quyết định cần gọi tool nào, với tham số gì |
| 13 | Host | Hỏi người dùng đồng ý (human-in-the-loop) |
| 14 | Client | Gửi `tools/call` với `name` và `arguments` |
| 15 | Server | Thực thi (gọi API, đọc DB, ghi file...) |
| 16 | Server | Trả `result` gồm `content` (text/image/...) và `isError` |
| 17 | Host | Chuyển kết quả cho LLM như một phần ngữ cảnh |
| 18 | LLM | Tổng hợp và trả lời người dùng |

Vòng 12 → 17 có thể lặp nhiều lần: LLM gọi nhiều tool liên tiếp, hoặc gọi tool rồi dựa vào kết quả để gọi tool tiếp theo.

### 9.3. Luồng cập nhật động

```
 Server thay đổi danh sách tool (ví dụ: bật thêm tính năng)
   │
   ├── gửi notifications/tools/list_changed ──► Client
   │                                             │
   │                                             ├── gọi lại tools/list
   │                                             └── host cập nhật ngữ cảnh LLM
```

Tương tự với `notifications/resources/updated` (khi resource đã subscribe thay đổi).

### 9.4. Luồng khi server cần LLM (Sampling)

```
 Server ── sampling/createMessage ──► Client ──► Host (duyệt, gọi LLM)
                                                    │
 Server ◄──────── kết quả sinh văn bản ◄───────────┘
```

### 9.5. Luồng Elicitation (hỏi người dùng)

```
 Server ── elicitation/create (câu hỏi + schema form) ──► Client
                                                           │
                                             Host hiện form cho người dùng
                                                           │
 Server ◄────── dữ liệu người dùng nhập (hoặc từ chối) ◄──┘
```

---

## 10. Xác thực và bảo mật

### 10.1. Xác thực

- **stdio:** thường không dùng OAuth; server kế thừa quyền của người dùng trên máy, hoặc nhận token qua biến môi trường.
- **Streamable HTTP:** đặc tả khuyến nghị **OAuth 2.1** với PKCE:
  - **Authorization server:** cấp token.
  - **Resource server (MCP server):** kiểm tra token trước khi phục vụ.
  - **Protected resource metadata** (RFC 9728): server công bố nó cần token từ đâu.
- Quy tắc quan trọng: server **không được chuyển tiếp (passthrough) token** mà nó nhận từ client sang API khác một cách mù quáng. Mỗi server nên có token riêng, đúng audience.

### 10.2. Các rủi ro cần lưu ý

| Rủi ro | Mô tả | Biện pháp |
|---|---|---|
| **Prompt injection** | Nội dung từ resource/tool chứa lệnh giả dạng chỉ dẫn cho LLM | Không tin tuyệt đối vào dữ liệu ngoài; cần consent |
| **Tool poisoning** | Mô tả tool bị sửa để lừa LLM | Chỉ cài server đáng tin; xem xét mô tả tool |
| **Tool có tác dụng phụ** | Tool xóa/ghi dữ liệu bị gọi ngoài ý muốn | Human-in-the-loop, annotation `destructiveHint` |
| **Quyền quá rộng** | Server có quyền truy cập toàn bộ hệ thống | Nguyên tắc least privilege, giới hạn roots |
| **Lộ dữ liệu** | Server gửi dữ liệu nhạy cảm ra ngoài | Kiểm tra server, không cấp dữ liệu không cần thiết |

**Nguyên tắc cốt lõi của đặc tả:** người dùng phải được **đồng ý rõ ràng** trước khi dữ liệu được truy cập và trước khi công cụ được thực thi; host là nơi thực thi các quyết định này.

---

## 11. Cấu hình MCP trong thực tế

### 11.1. Claude Desktop (`claude_desktop_config.json`)

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

- `command` + `args`: lệnh khởi chạy server (stdio).
- `env`: biến môi trường (ví dụ API key).

### 11.2. Claude Code (CLI)

```bash
# Thêm server stdio
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem D:\VIBECODE

# Thêm server HTTP từ xa
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Liệt kê
claude mcp list
```

Trong phiên tương tác, dùng `/mcp` để xem trạng thái và xác thực.

### 11.3. VS Code (`.vscode/mcp.json`)

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

> Lưu ý: khóa gốc của VS Code là `servers`, còn Claude Desktop dùng `mcpServers`.

### 11.4. Phạm vi cấu hình

| Phạm vi | Ý nghĩa |
|---|---|
| Local / Project / User (Claude Code) | Chỉ cho mình bạn, theo dự án chia sẻ, hoặc cho mọi dự án |
| Workspace (VS Code) | Theo thư mục dự án |

---

## 12. Tạo một MCP server đơn giản

Ví dụ dùng **Python SDK** (`mcp`), dùng `FastMCP`:

```python
# server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("ví-dụ-server")

@mcp.tool()
def cộng(a: int, b: int) -> int:
    """Cộng hai số nguyên."""
    return a + b

@mcp.resource("note://{ten}")
def đọc_ghi_chú(ten: str) -> str:
    """Đọc một ghi chú theo tên."""
    return f"Nội dung ghi chú {ten}"

@mcp.prompt()
def tóm_tắt(văn_bản: str) -> str:
    """Mẫu prompt tóm tắt."""
    return f"Hãy tóm tắt ngắn gọn đoạn sau:\n\n{văn_bản}"

if __name__ == "__main__":
    mcp.run()  # mặc định dùng transport stdio
```

Đăng ký với Claude Code:

```bash
claude mcp add ví-dụ -- python server.py
```

**Kiểm thử bằng MCP Inspector:**

```bash
npx @modelcontextprotocol/inspector python server.py
```

Inspector mở giao diện web để xem danh sách tool/resource/prompt và gọi thử từng cái, rất hữu ích khi debug.

---

## 13. Bảng thuật ngữ (Glossary)

| Thuật ngữ | Tiếng Việt / Giải thích |
|---|---|
| **MCP** (Model Context Protocol) | Giao thức mở kết nối AI với công cụ và dữ liệu bên ngoài |
| **Host** | Ứng dụng AI mà người dùng dùng; chứa LLM và các client |
| **Client** | Thành phần trong host, kết nối 1:1 với một server |
| **Server** | Chương trình cung cấp tools/resources/prompts |
| **Primitive** | Loại khả năng cơ bản: tools, resources, prompts, sampling, elicitation, roots |
| **Tool** | Hàm LLM có thể gọi để hành động hoặc truy vấn |
| **Resource** | Dữ liệu chỉ đọc, định danh bằng URI |
| **Resource template** | URI có tham số, ví dụ `note://{tên}` |
| **Prompt** | Mẫu lệnh có sẵn, người dùng chọn để dùng |
| **Sampling** | Server nhờ LLM của host sinh nội dung |
| **Elicitation** | Server hỏi thêm thông tin từ người dùng |
| **Roots** | Phạm vi thư mục/URI client cho server được làm việc |
| **Capability** | Khả năng một bên hỗ trợ, khai báo lúc initialize |
| **Initialize / Handshake** | Bước bắt tay đầu phiên, đàm phán phiên bản và khả năng |
| **protocolVersion** | Phiên bản đặc tả MCP đang dùng, dạng ngày |
| **JSON-RPC 2.0** | Định dạng tin nhắn request/response/notification nền tảng của MCP |
| **Request / Response / Notification** | Ba loại tin nhắn JSON-RPC; notification không cần trả lời |
| **Transport** | Kênh vận chuyển tin nhắn: stdio, Streamable HTTP |
| **stdio** | Giao tiếp qua stdin/stdout của tiến trình con, dùng cho server cục bộ |
| **Streamable HTTP** | Transport HTTP cho server từ xa, có thể stream bằng SSE |
| **SSE** (Server-Sent Events) | Cơ chế server đẩy dữ liệu về client qua HTTP |
| **inputSchema** | JSON Schema mô tả tham số đầu vào của tool |
| **outputSchema / structuredContent** | Mô tả và dữ liệu có cấu trúc của kết quả tool |
| **content** | Mảng nội dung kết quả: text, image, audio, resource link... |
| **isError** | Cờ báo kết quả tool là lỗi |
| **Annotation** | Gợi ý về tool như `readOnlyHint`, `destructiveHint` |
| **Notification (list_changed)** | Thông báo danh sách tool/resource/prompt đã thay đổi |
| **Subscribe** | Đăng ký nhận thông báo khi một resource thay đổi |
| **Progress / Cancellation** | Báo tiến độ / huỷ một request đang chạy |
| **Pagination** | Chia danh sách lớn thành nhiều trang |
| **Human-in-the-loop** | Người dùng phải xác nhận trước khi công cụ chạy |
| **Consent** | Sự đồng ý rõ ràng của người dùng về truy cập dữ liệu/công cụ |
| **OAuth 2.1 / PKCE** | Chuẩn xác thực dùng cho transport HTTP |
| **Authorization server** | Dịch vụ cấp token truy cập |
| **Resource server** | MCP server kiểm tra token trước khi phục vụ |
| **Prompt injection** | Tấn công nhúng lệnh độc vào dữ liệu để điều khiển LLM |
| **Tool poisoning** | Tấn công sửa mô tả tool để lừa LLM |
| **Least privilege** | Nguyên tắc chỉ cấp quyền tối thiểu cần thiết |
| **MCP Inspector** | Công cụ web để kiểm thử và debug MCP server |
| **SDK** | Bộ thư viện chính thức để xây client/server (TypeScript, Python, ...) |
| **Agent** | Hệ thống AI tự lặp: suy nghĩ → gọi tool → quan sát kết quả → tiếp tục |

---

## 14. Tóm tắt

1. **MCP** là chuẩn chung để AI kết nối với công cụ và dữ liệu, giúp tránh tích hợp rời rạc N×M.
2. **Ba vai trò:** Host (ứng dụng AI) chứa **Client** (kết nối 1:1), client nói chuyện với **Server** (cung cấp khả năng).
3. **Ba primitive chính phía server:** **Tools** (model gọi), **Resources** (dữ liệu chỉ đọc), **Prompts** (mẫu lệnh do người dùng chọn).
4. **Ba primitive phía client:** **Sampling**, **Elicitation**, **Roots**.
5. **Giao thức:** JSON-RPC 2.0, song hướng; **transport** là stdio (cục bộ) hoặc Streamable HTTP (từ xa).
6. **Flow:** khởi chạy/kết nối → `initialize` → `initialized` → `tools/list` → LLM quyết định → người dùng đồng ý → `tools/call` → kết quả đưa lại cho LLM → câu trả lời.
7. **Bảo mật:** consent của người dùng, least privilege, cảnh giác prompt injection và tool poisoning, OAuth 2.1 cho HTTP.

---

> **Nguồn tham khảo:** đặc tả chính thức tại modelcontextprotocol.io (kiểm tra phiên bản mới nhất vì đặc tả được cập nhật định kỳ). Các chi tiết về tên method và phiên bản trong tài liệu này nên được đối chiếu với đặc tả hiện hành khi triển khai thực tế.
