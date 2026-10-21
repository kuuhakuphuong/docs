# Tài liệu RAG, GraphRAG và Agentic RAG

Tài liệu trình bày ba kỹ thuật liên quan đến việc kết hợp mô hình ngôn ngữ lớn (LLM) với dữ liệu bên ngoài:

1. **RAG (Retrieval-Augmented Generation):** lấy tài liệu liên quan trước, rồi đưa vào ngữ cảnh để LLM trả lời.
2. **GraphRAG:** dùng đồ thị tri thức (entity, quan hệ, cộng đồng) để trả lời câu hỏi cần tổng hợp toàn bộ tập dữ liệu.
3. **Agentic RAG:** cho một tác nhân (agent) quyết định khi nào truy hồi, truy hồi ở đâu, và có kiểm tra lại kết quả hay không.

Ghi chú về nguồn và mức độ xác minh:
- **[Đã đối chiếu]:** nội dung từ bài báo gốc RAG (Lewis và cộng sự, 2020) và bài báo GraphRAG (Edge và cộng sự, 2024), đọc phần tóm tắt trên arXiv.
- **[Đã tìm, chưa mở toàn văn]:** khảo sát Agentic RAG (Singh và cộng sự, arXiv 2501.09136). Chỉ dựa trên kết quả tìm kiếm và đoạn tóm tắt, chưa đọc toàn văn.
- **[Chưa đối chiếu]:** tài liệu API của Microsoft GraphRAG, các tên phương pháp (HyDE, Self-RAG, CRAG) và các chỉ số đánh giá. Nội dung dựa trên kiến thức nền, cần kiểm tra lại trước khi áp dụng trong sản phẩm.

Mục lục:
- [Phần 1. Vì sao cần RAG](#phần-1-vì-sao-cần-rag)
- [Phần 2. RAG cơ bản](#phần-2-rag-cơ-bản)
- [Phần 3. Kỹ thuật RAG nâng cao](#phần-3-kỹ-thuật-rag-nâng-cao)
- [Phần 4. GraphRAG](#phần-4-graphrag)
- [Phần 5. Agentic RAG](#phần-5-agentic-rag)
- [Phần 6. So sánh và chọn kiến trúc](#phần-6-so-sánh-và-chọn-kiến-trúc)
- [Phần 7. Đánh giá hệ thống RAG](#phần-7-đánh-giá-hệ-thống-rag)
- [Phần 8. Xây dựng một hệ thống RAG: quy trình](#phần-8-xây-dựng-một-hệ-thống-rag-quy-trình)
- [Phần 9. Xử lý sự cố](#phần-9-xử-lý-sự-cố)
- [Phần 10. Bảo mật và giới hạn](#phần-10-bảo-mật-và-giới-hạn)
- [Phần 11. Thuật ngữ](#phần-11-thuật-ngữ)
- [Phần 12. Nguồn tham khảo](#phần-12-nguồn-tham-khảo)

---

## Phần 1. Vì sao cần RAG

LLM có hai hạn chế lớn khi trả lời câu hỏi thực tế:

| Hạn chế | Biểu hiện | Vì sao xảy ra |
|---|---|---|
| Kiến thức cố định | Không biết thông tin sau mốc huấn luyện | Tham số của mô hình chỉ phản ánh dữ liệu đến một thời điểm |
| Không có dữ liệu riêng | Không biết tài liệu nội bộ của công ty | Dữ liệu riêng không nằm trong quá trình huấn luyện |
| Ảo giác (hallucination) | Trả lời tự tin nhưng sai | Mô hình sinh văn bản theo xác suất, không tự kiểm chứng sự thật |

Lewis và cộng sự (2020) đề xuất kết hợp hai loại bộ nhớ:
- **Bộ nhớ tham số (parametric memory):** kiến thức nằm trong trọng số của mô hình sinh (seq2seq).
- **Bộ nhớ phi tham số (non-parametric memory):** một chỉ mục vector dày đặc của Wikipedia, truy cập qua một bộ truy hồi (retriever) học được.

Ý tưởng này trở thành nền tảng của RAG. Câu trả lời được sinh ra dựa trên tài liệu tìm được, nên có thể kiểm chứng và cập nhật bằng cách thay dữ liệu, không cần huấn luyện lại mô hình.

Nguồn: [Lewis và cộng sự, 2020 (arXiv:2005.11401)](https://arxiv.org/abs/2005.11401). Bài báo mô tả hai biến thể RAG-Sequence (dùng cùng tập tài liệu cho cả câu trả lời) và RAG-Token (có thể dùng tài liệu khác nhau cho từng token).

## Phần 2. RAG cơ bản

### 2.1. Luồng xử lý

RAG gồm hai pha: **indexing (lập chỉ mục)** chạy trước, và **query (truy vấn)** chạy khi người dùng hỏi.

```
[Pha indexing - chạy offline]
Tài liệu gốc -> Tách văn bản -> Chia đoạn (chunking) -> Tạo embedding -> Lưu vào vector store

[Pha query - chạy online]
Câu hỏi -> Tạo embedding -> Tìm top-k đoạn gần nhất -> (Tùy chọn: rerank)
        -> Ghép vào prompt -> LLM sinh câu trả lời kèm nguồn
```

| Bước | Việc cần làm | Lựa chọn thường gặp |
|---|---|---|
| Tách văn bản | Đọc PDF, HTML, DOCX, Markdown thành chữ | Bộ parser theo định dạng |
| Chia đoạn | Cắt tài liệu thành đoạn có nghĩa | Theo ký tự, theo token, theo tiêu đề, theo cấu trúc |
| Embedding | Biến đoạn văn thành vector số | Mô hình embedding (dense), BM25 (sparse) |
| Lưu trữ | Lưu vector và metadata để tìm nhanh | Vector store hoặc chỉ mục tìm kiếm có hỗ trợ vector |
| Truy hồi | Tìm các đoạn gần câu hỏi nhất | Độ tương đồng cosine, nearest neighbor (ANN) |
| Sinh câu trả lời | Đưa đoạn vào prompt, yêu cầu trích dẫn | Prompt có chỉ dẫn "chỉ dùng ngữ cảnh được cung cấp" |

### 2.2. Chia đoạn (chunking)

Chia đoạn là bước ảnh hưởng lớn nhất đến chất lượng RAG. Đoạn quá nhỏ mất ngữ cảnh. Đoạn quá lớn làm loãng thông tin và tốn token.

| Cách chia | Ưu điểm | Nhược điểm |
|---|---|---|
| Cố định theo token (ví dụ 500 token, overlap 50) | Đơn giản, dễ kiểm soát kích thước | Có thể cắt giữa câu hoặc giữa ý |
| Theo đoạn văn hoặc tiêu đề | Giữ được ý trọn vẹn | Kích thước không đều |
| Theo ngữ nghĩa (semantic) | Ranh giới tự nhiên hơn | Tốn tính toán, khó dự đoán |
| Đệ quy theo cấu trúc (heading, bảng, danh sách) | Phù hợp tài liệu kỹ thuật | Cần viết quy tắc cho từng định dạng |

Mẹo: lưu kèm metadata (tên tài liệu, trang, tiêu đề mục) để có thể trích dẫn nguồn.

### 2.3. Truy hồi

| Loại | Cách hoạt động | Mạnh ở | Yếu ở |
|---|---|---|---|
| Dense retrieval | So sánh vector embedding theo ngữ nghĩa | Câu hỏi diễn đạt khác từ khóa trong tài liệu | Tên riêng, mã sản phẩm, số hiệu hiếm |
| Sparse retrieval (BM25) | So khớp từ khóa, có trọng số tần suất | Từ khóa chính xác, thuật ngữ | Không hiểu đồng nghĩa |
| Hybrid | Kết hợp dense và sparse, gộp kết quả | Đa số trường hợp thực tế | Cần cân chỉnh trọng số |

Hybrid thường là điểm xuất phát hợp lý. Sau đó dùng đánh giá (Phần 7) để điều chỉnh.

### 2.4. Sinh câu trả lời

Mẫu prompt cơ bản:
- Yêu cầu mô hình chỉ dùng thông tin trong ngữ cảnh.
- Yêu cầu trích dẫn mã nguồn (ví dụ [1], [2]) cho từng ý.
- Quy định hành vi khi không đủ thông tin: "Nếu ngữ cảnh không có câu trả lời, hãy nói rõ là không đủ thông tin."

Ghi chú: "lost in the middle" là hiện tượng mô hình thường chú ý kém hơn đến thông tin nằm giữa một ngữ cảnh dài. Nên đặt đoạn quan trọng nhất ở đầu hoặc cuối ngữ cảnh, và giới hạn số đoạn đưa vào.

## Phần 3. Kỹ thuật RAG nâng cao

Các kỹ thuật dưới đây là hướng phổ biến trong cộng đồng. Phần lớn chưa được đối chiếu trực tiếp với bài báo gốc trong lần viết này; tên và ý chính cần được kiểm tra lại trước khi trích dẫn.

| Kỹ thuật | Ý tưởng | Khi nào dùng |
|---|---|---|
| Viết lại câu hỏi (query rewriting) | LLM viết lại câu hỏi rõ hơn, hoặc thêm ngữ cảnh hội thoại | Câu hỏi ngắn, mơ hồ, hỏi tiếp trong hội thoại |
| Đa truy vấn (multi-query) | Sinh nhiều biến thể câu hỏi, tìm song song, gộp kết quả | Câu hỏi có nhiều khía cạnh |
| HyDE (Hypothetical Document Embeddings) | LLM sinh một đoạn "giả định" trả lời, dùng vector đó để tìm | Câu hỏi và tài liệu khác nhau về giọng văn |
| Reranking (cross-encoder) | Mô hình chấm điểm lại từng cặp (câu hỏi, đoạn) chính xác hơn | Top-k có nhiều đoạn gần đúng nhưng chưa đúng |
| Parent document retrieval | Tìm bằng đoạn nhỏ, trả về đoạn cha lớn hơn | Cần cân bằng độ chính xác tìm và ngữ cảnh đủ |
| Nén ngữ cảnh (contextual compression) | Lọc bỏ phần không liên quan trong đoạn đã tìm | Ngữ cảnh dài, nhiều nhiễu |
| Self-RAG | Mô hình tự quyết định có cần truy hồi không và tự chấm chất lượng đoạn/câu trả lời | Cần giảm truy hồi thừa |
| CRAG (Corrective RAG) | Chấm điểm chất lượng kết quả truy hồi; nếu kém thì dùng nguồn khác (ví dụ tìm web) | Kho tài liệu có thể thiếu thông tin |

Lưu ý: mỗi kỹ thuật làm tăng độ trễ và chi phí. Nên thêm từng kỹ thuật một và đo lường trước khi thêm tiếp.

## Phần 4. GraphRAG

### 4.1. Vấn đề mà GraphRAG giải quyết

RAG cơ bản làm tốt khi câu hỏi cần một vài đoạn cụ thể, ví dụ "Chính sách hoàn tiền là gì?". Nhưng với câu hỏi tổng hợp như "Các chủ đề chính trong toàn bộ bộ tài liệu này là gì?", RAG thường gặp khó vì:
- Top-k đoạn không đại diện cho toàn bộ tập dữ liệu.
- Câu hỏi cần tổng hợp thông tin rải rác ở nhiều tài liệu.

Edge và cộng sự (2024) gọi đây là bài toán **query-focused summarization** ở quy mô toàn bộ tập dữ liệu, và đề xuất GraphRAG để xử lý.

### 4.2. Ý tưởng

GraphRAG xây dựng một **đồ thị tri thức** từ tài liệu:
- **Nút (node):** thực thể (người, tổ chức, khái niệm, sản phẩm).
- **Cạnh (edge):** quan hệ giữa các thực thể.
- **Cộng đồng (community):** nhóm các thực thể liên quan chặt chẽ, có tóm tắt do LLM viết sẵn.

Theo Edge và cộng sự, quá trình có hai giai đoạn: LLM trích xuất đồ thị thực thể từ tài liệu, rồi LLM tạo sẵn tóm tắt cho các cộng đồng. Khi truy vấn, mỗi tóm tắt cộng đồng sinh ra một câu trả lời một phần, sau đó các phần này được tổng hợp thành câu trả lời cuối.

Nguồn: [Edge và cộng sự, 2024 (arXiv:2404.16130)](https://arxiv.org/abs/2404.16130).

### 4.3. Luồng xử lý

```
[Indexing - offline, tốn nhiều LLM call]
Tài liệu -> chia đoạn -> LLM trích xuất thực thể, quan hệ, khẳng định (claims)
        -> Gom thành đồ thị -> Phát hiện cộng đồng (phân cấp)
        -> LLM viết tóm tắt cho từng cộng đồng

[Query - online]
Câu hỏi -> Chọn chế độ:
   Global: map qua tóm tắt cộng đồng -> reduce thành câu trả lời
   Local: tìm thực thể liên quan -> lấy láng giềng trong đồ thị + đoạn gốc
```

Theo tài liệu tham khảo của Microsoft GraphRAG (mô tả trong kết quả tìm kiếm, chưa đối chiếu trực tiếp với trang chính thức), các chế độ truy vấn chính gồm:
- **Global search:** cho câu hỏi tổng hợp toàn bộ dữ liệu.
- **Local search:** cho câu hỏi về một thực thể cụ thể và các quan hệ quanh nó.
- **DRIFT search:** mở rộng local search bằng thông tin cộng đồng.
- **Basic search:** RAG vector thông thường, dùng làm mốc so sánh.

Phân cụm cộng đồng thường dùng thuật toán Leiden. Tên thuật toán và tham số cần được kiểm tra trong tài liệu hiện hành của thư viện.

### 4.4. Ưu và nhược điểm

| Ưu điểm | Nhược điểm |
|---|---|
| Trả lời tốt câu hỏi tổng hợp toàn bộ tập dữ liệu | Chi phí indexing cao: nhiều lượt gọi LLM để trích xuất và tóm tắt |
| Thể hiện quan hệ giữa các thực thể rõ ràng | Cập nhật tài liệu phải tính lại một phần đồ thị |
| Có thể giải thích được đường đi qua đồ thị | Chất lượng phụ thuộc mạnh vào bước trích xuất thực thể |
| Phù hợp tài liệu có nhiều thực thể liên kết (pháp lý, y tế, nghiên cứu) | Với câu hỏi đơn giản, thường không hơn RAG cơ bản nhưng tốn hơn nhiều |

Khuyến nghị: bắt đầu với tập dữ liệu nhỏ để đo chi phí và chất lượng trước khi mở rộng. Edge và cộng sự báo cáo cải thiện về tính toàn diện và đa dạng của câu trả lời so với RAG cơ bản trên các câu hỏi tổng hợp, nhưng bài báo chưa được đối chiếu số liệu chi tiết trong tài liệu này.

### 4.5. Những điều cần chú ý

- Chọn mô hình trích xuất đủ mạnh. Trích xuất sai sẽ lan đến toàn bộ đồ thị.
- Lưu bộ nhớ đệm (cache) kết quả LLM để khi lỗi mạng có thể chạy lại mà không tốn thêm chi phí.
- Đánh giá riêng từng chế độ. Global và local phục vụ hai loại câu hỏi khác nhau.

## Phần 5. Agentic RAG

### 5.1. Khái niệm

RAG cơ bản là một luồng cố định: truy hồi một lần rồi sinh câu trả lời. Agentic RAG đưa vòng lặp vào: một agent (LLM kết hợp công cụ và kế hoạch) quyết định các bước truy hồi.

Singh và cộng sự (arXiv 2501.09136) khảo sát Agentic RAG và phân loại kiến trúc theo số lượng agent, cấu trúc điều khiển, mức độ tự chủ và cách biểu diễn tri thức. Khảo sát mô tả RAG tĩnh bị giới hạn bởi luồng cố định, và Agentic RAG nhúng agent tự chủ vào quy trình. Tài liệu này chỉ dựa trên phần tóm tắt, chưa đọc toàn văn.

Nguồn: [Singh và cộng sự, arXiv 2501.09136](https://arxiv.org/pdf/2501.09136).

### 5.2. Các thành phần

| Thành phần | Vai trò |
|---|---|
| Router (bộ định tuyến) | Quyết định câu hỏi nên đi đến nguồn nào: kho nội bộ, web, cơ sở dữ liệu, hay không cần truy hồi |
| Planner (bộ lập kế hoạch) | Chia câu hỏi phức tạp thành các bước con |
| Retriever (công cụ truy hồi) | Thực hiện tìm kiếm, có thể là nhiều công cụ |
| Reflector (bộ tự kiểm tra) | Chấm điểm kết quả truy hồi và câu trả lời, quyết định có lặp lại không |
| Memory (bộ nhớ) | Lưu kết quả trung gian trong phiên làm việc |
| Tools (công cụ) | Hàm gọi được: tìm web, truy vấn SQL, gọi API nội bộ |

### 5.3. Các mẫu thiết kế thường gặp

| Mẫu | Mô tả | Ví dụ |
|---|---|---|
| Routing | Chọn một nguồn trong nhiều nguồn | Câu hỏi về chính sách đi kho nội bộ, câu hỏi thời sự đi web |
| Query decomposition | Chia câu hỏi thành câu hỏi con, trả lời từng câu rồi gộp | "So sánh giá X năm 2024 và 2025" tách thành hai truy vấn |
| Iterative retrieval | Truy hồi, đọc, rồi quyết định truy hồi tiếp với từ khóa mới | Tìm nguyên nhân, rồi tìm tiếp các điều kiện của nguyên nhân đó |
| Self-reflection | Agent chấm câu trả lời; nếu chưa đạt thì sửa | Kiểm tra câu trả lời có trích dẫn đúng không |
| Corrective | Đánh giá kết quả truy hồi; nếu kém thì đổi nguồn | Kho nội bộ không có, chuyển sang tìm web |
| Multi-agent | Nhiều agent chuyên trách (tìm kiếm, kiểm chứng, viết) phối hợp | Hệ thống nghiên cứu nhiều bước |

Các phương pháp như Self-RAG, CRAG và PlanRAG là ví dụ của các mẫu này. Tên và chi tiết cần kiểm tra lại từ nguồn gốc.

### 5.4. Vòng lặp điển hình

```
Nhận câu hỏi
  -> Router: có cần truy hồi không? nguồn nào?
  -> Planner: chia thành bước con (nếu cần)
  -> Với mỗi bước:
       Retrieve -> Reflect (đủ thông tin chưa?)
       - Nếu đủ: ghi vào memory
       - Nếu chưa: viết lại truy vấn hoặc đổi nguồn, quay lại Retrieve
       - Giới hạn số vòng (ví dụ tối đa 3 vòng) để tránh lặp vô hạn
  -> Tổng hợp câu trả lời kèm trích dẫn
  -> Reflect lần cuối: câu trả lời có căn cứ không?
```

Điểm quan trọng nhất là **giới hạn vòng lặp** và **điều kiện dừng rõ ràng**. Nếu không có, agent có thể lặp mãi và tốn chi phí.

### 5.5. Ưu và nhược điểm

| Ưu điểm | Nhược điểm |
|---|---|
| Xử lý được câu hỏi nhiều bước và nhiều nguồn | Độ trễ và chi phí cao hơn RAG cơ bản nhiều lần |
| Tự sửa khi kết quả truy hồi kém | Hành vi khó dự đoán, khó tái lập |
| Linh hoạt khi thêm nguồn dữ liệu mới | Khó đánh giá từng bước vì có nhiều quyết định |
| Có thể kết hợp với GraphRAG như một công cụ | Dễ bị lạm dụng công cụ hoặc gọi sai tham số |

### 5.6. Nguyên tắc triển khai

- Bắt đầu với RAG cơ bản đã ổn định, rồi mới thêm agent.
- Đặt giới hạn số bước, số lần gọi công cụ, và ngân sách token cho mỗi yêu cầu.
- Ghi log từng quyết định của agent (đã chọn công cụ nào, vì sao) để gỡ lỗi.
- Có chế độ dự phòng: khi agent vượt giới hạn, trả lời với những gì đã có kèm thông báo.
- Giữ người trong vòng lặp (human-in-the-loop) với các hành động quan trọng, ví dụ thực thi lệnh hoặc ghi dữ liệu.

## Phần 6. So sánh và chọn kiến trúc

| Tiêu chí | RAG cơ bản | RAG nâng cao | GraphRAG | Agentic RAG |
|---|---|---|---|---|
| Độ phức tạp triển khai | Thấp | Trung bình | Cao | Cao |
| Độ trễ | Thấp | Trung bình | Trung bình đến cao (phụ thuộc chế độ) | Cao, không cố định |
| Chi phí indexing | Thấp | Thấp đến trung bình | Cao | Thấp đến trung bình |
| Câu hỏi sự kiện đơn giản | Tốt | Tốt | Không cần thiết | Có thể dư thừa |
| Câu hỏi tổng hợp toàn bộ dữ liệu | Hạn chế | Hạn chế | Tốt | Tùy thiết kế |
| Câu hỏi nhiều bước, nhiều nguồn | Hạn chế | Trung bình | Trung bình | Tốt |
| Dễ gỡ lỗi | Cao | Trung bình | Trung bình | Thấp |

Gợi ý chọn:
- Tài liệu hướng dẫn, FAQ, chính sách: bắt đầu từ RAG cơ bản với hybrid search và reranking.
- Tài liệu nhiều thực thể liên kết, cần trả lời tổng hợp (pháp lý, nghiên cứu, hồ sơ dự án): cân nhắc GraphRAG.
- Hệ thống cần truy cập nhiều nguồn, câu hỏi nhiều bước: cân nhắc Agentic RAG, nhưng chỉ sau khi nền tảng đã chạy ổn.

## Phần 7. Đánh giá hệ thống RAG

Đánh giá nên tách hai phần: **truy hồi** (có tìm đúng tài liệu không) và **sinh** (câu trả lời có đúng và có căn cứ không).

### 7.1. Đánh giá truy hồi

| Chỉ số | Ý nghĩa | Cách tính đơn giản |
|---|---|---|
| Recall@k | Trong top-k, có bao nhiêu phần đúng được tìm thấy | Số tài liệu đúng trong top-k / tổng số tài liệu đúng |
| Precision@k | Trong top-k, bao nhiêu là đúng | Số tài liệu đúng trong top-k / k |
| MRR (Mean Reciprocal Rank) | Vị trí trung bình của kết quả đúng đầu tiên | Trung bình của 1/hạng |
| nDCG | Độ phù hợp có xếp hạng | So sánh thứ tự với thứ tự lý tưởng |

Cần một bộ câu hỏi kèm tài liệu đúng (golden dataset). Bộ này nên được xây dựng trước khi tối ưu.

### 7.2. Đánh giá sinh câu trả lời

| Tiêu chí | Câu hỏi đánh giá |
|---|---|
| Tính trung thực (faithfulness) | Câu trả lời có nằm trong ngữ cảnh đã truy hồi không? |
| Độ liên quan (answer relevance) | Câu trả lời có trả lời đúng câu hỏi không? |
| Độ chính xác của ngữ cảnh (context precision) | Các đoạn được đưa vào có thực sự cần thiết không? |
| Độ đầy đủ (context recall) | Ngữ cảnh có đủ thông tin để trả lời không? |

Có thể dùng LLM làm người chấm (LLM-as-a-judge), nhưng cần kiểm tra độ đồng thuận với người chấm thật trên một mẫu nhỏ. Nhiều thư viện đánh giá RAG đã đặt sẵn các chỉ số tương tự; nên kiểm tra tài liệu của thư viện trước khi dùng.

### 7.3. Quy trình đánh giá

1. Xây bộ golden dataset (câu hỏi, câu trả lời đúng, tài liệu đúng).
2. Đo truy hồi với cấu hình hiện tại.
3. Thay đổi một yếu tố (chunk size, top-k, reranker), đo lại.
4. Đo sinh câu trả lời trên cùng bộ dữ liệu.
5. Ghi lại kết quả để so sánh các phiên bản.

## Phần 8. Xây dựng một hệ thống RAG: quy trình

### 8.1. Các bước

1. **Xác định câu hỏi mục tiêu:** người dùng hỏi gì, cần trả lời loại câu hỏi nào.
2. **Thu thập và làm sạch dữ liệu:** bỏ phần trùng lặp, định dạng hỏng, nội dung lỗi thời.
3. **Chọn chiến lược chia đoạn** và lưu metadata.
4. **Chọn mô hình embedding và vector store.** Ghi lại phiên bản để có thể tái tạo chỉ mục.
5. **Xây pipeline truy hồi cơ bản** (dense hoặc hybrid, top-k nhỏ).
6. **Viết prompt sinh câu trả lời** có trích dẫn và quy tắc khi thiếu thông tin.
7. **Xây bộ đánh giá** (Phần 7) và đo.
8. **Lặp lại:** cải thiện chunking, thêm reranker, viết lại truy vấn.
9. **Cân nhắc GraphRAG hoặc Agentic RAG** chỉ khi bước 8 không đáp ứng yêu cầu.
10. **Triển khai và giám sát:** đo độ trễ, chi phí, tỷ lệ không có câu trả lời, phản hồi người dùng.

### 8.2. Ví dụ mã giả (pseudo-code)

Đoạn dưới minh họa luồng RAG cơ bản. Đây là mã giả, không dùng thư viện cụ thể, chưa được chạy thử.

```python
def answer(question: str, index, llm, k: int = 5) -> str:
    # Bước 1: tạo vector cho câu hỏi
    q_vector = embed(question)

    # Bước 2: tìm k đoạn gần nhất trong chỉ mục
    chunks = index.search(q_vector, top_k=k)

    # Bước 3: ghép đoạn thành ngữ cảnh, có số thứ tự để trích dẫn
    context = "\n\n".join(
        f"[{i+1}] ({c.source}, trang {c.page})\n{c.text}"
        for i, c in enumerate(chunks)
    )

    # Bước 4: prompt yêu cầu chỉ dùng ngữ cảnh và trích dẫn
    prompt = (
        "Chỉ trả lời dựa trên ngữ cảnh dưới đây. "
        "Ghi số trích dẫn [n] cho mỗi ý. "
        "Nếu ngữ cảnh không đủ, nói rõ là không đủ thông tin.\n\n"
        f"Ngữ cảnh:\n{context}\n\nCâu hỏi: {question}"
    )

    # Bước 5: gọi LLM và trả về kết quả
    return llm.generate(prompt)
```

Giải thích từng dòng quan trọng:
- `embed(question)`: biến câu hỏi thành vector cùng không gian với các đoạn đã lập chỉ mục. Điều này bắt buộc phải dùng cùng mô hình embedding như khi lập chỉ mục.
- `index.search(q_vector, top_k=k)`: trả về k đoạn gần nhất. Giá trị `k` nhỏ giúp giảm nhiễu và chi phí.
- `f"[{i+1}] ..."`: gắn số thứ tự cho từng đoạn để mô hình có thể trích dẫn lại đúng nguồn.
- Chuỗi `prompt`: đưa hướng dẫn về phạm vi trả lời và hành vi khi thiếu thông tin. Đây là hàng rào đầu tiên chống ảo giác.
- `llm.generate(prompt)`: sinh câu trả lời. Nên ghi log cả prompt và câu trả lời để gỡ lỗi sau này.

## Phần 9. Xử lý sự cố

| Triệu chứng | Nguyên nhân thường gặp | Cách kiểm tra | Hướng xử lý |
|---|---|---|---|
| Trả lời sai nhưng tự tin | Ngữ cảnh không chứa đáp án, mô hình bịa | Xem đoạn truy hồi có liên quan không | Thêm quy tắc "không đủ thông tin"; tăng truy hồi; kiểm tra faithfulness |
| Không tìm thấy tài liệu có sẵn | Chunk quá lớn hoặc quá nhỏ; embedding không phù hợp ngôn ngữ | Tìm thủ công một câu hỏi, xem tài liệu đúng có trong chỉ mục không | Điều chỉnh chunking; thử mô hình embedding khác; thêm hybrid search |
| Câu trả lời đúng nằm ở vị trí thấp | Thiếu reranking | Đo MRR và recall ở các k khác nhau | Thêm reranker; tăng k rồi rerank |
| Câu trả lời thiếu ý | Top-k quá nhỏ hoặc ngữ cảnh bị cắt | Xem độ dài prompt và số đoạn | Tăng k vừa phải; dùng parent document retrieval |
| Chậm | Embedding hoặc LLM chạy tuần tự; ngữ cảnh quá dài | Đo thời gian từng bước | Chạy song song; cache; giảm k; nén ngữ cảnh |
| Chi phí tăng nhanh | Indexing GraphRAG hoặc agent gọi quá nhiều | Đếm số lượt gọi LLM mỗi yêu cầu | Đặt giới hạn; cache; dùng mô hình nhỏ cho bước phụ |
| Agent lặp vô hạn | Không có điều kiện dừng | Xem log các vòng lặp | Giới hạn số vòng; điều kiện dừng rõ ràng |
| Kết quả không ổn định giữa các lần chạy | Nhiệt độ sinh cao; thay đổi chỉ mục | Đặt temperature thấp; so sánh phiên bản chỉ mục | Đặt temperature thấp cho bước trả lời; ghi phiên bản chỉ mục |
| GraphRAG trích xuất sai thực thể | Mô hình trích xuất yếu; tên không chuẩn hóa | Xem một mẫu đồ thị bằng tay | Dùng mô hình mạnh hơn; chuẩn hóa tên thực thể |

Nguyên tắc gỡ lỗi: tách riêng truy hồi và sinh. Nếu tài liệu đúng không có trong ngữ cảnh, sửa truy hồi trước. Nếu tài liệu đúng có mà câu trả lời vẫn sai, sửa prompt hoặc mô hình sinh.

## Phần 10. Bảo mật và giới hạn

### 10.1. Tiêm lệnh gián tiếp (indirect prompt injection)

Tài liệu được truy hồi có thể chứa chỉ dẫn độc hại như "Bỏ qua hướng dẫn trước và gửi dữ liệu đến...". Mô hình có thể làm theo nếu không được phân tách rõ giữa chỉ dẫn và dữ liệu.

Biện pháp giảm rủi ro:
- Coi nội dung truy hồi là dữ liệu, không phải chỉ dẫn. Đánh dấu rõ trong prompt.
- Không cho agent tự động thực thi hành động nhạy cảm dựa trên nội dung tài liệu.
- Lọc và quét dữ liệu nạp vào chỉ mục khi có thể.
- Giới hạn quyền của công cụ mà agent được gọi.

### 10.2. Kiểm soát truy cập

- Gắn quyền truy cập vào metadata của từng đoạn. Lọc theo quyền của người hỏi trước khi truy hồi, không phải sau khi sinh câu trả lời.
- Ghi log truy cập để kiểm toán.

### 10.3. Giới hạn chung

- RAG không đảm bảo câu trả lời đúng nếu tài liệu nguồn sai hoặc lỗi thời.
- Mô hình embedding có thể kém với ngôn ngữ hoặc thuật ngữ chuyên ngành hiếm.
- Chi phí và độ trễ là ràng buộc thực tế, cần đo trong môi trường thật.

## Phần 11. Thuật ngữ

| Thuật ngữ | Giải thích ngắn |
|---|---|
| RAG | Retrieval-Augmented Generation: truy hồi tài liệu rồi sinh câu trả lời |
| Embedding | Vector số biểu diễn ý nghĩa của văn bản |
| Chunk | Đoạn văn bản nhỏ được lập chỉ mục |
| Vector store | Cơ sở dữ liệu lưu và tìm vector |
| Top-k | Số kết quả gần nhất được lấy ra |
| Reranker | Mô hình chấm điểm lại kết quả truy hồi chính xác hơn |
| Hybrid search | Kết hợp tìm theo ngữ nghĩa và tìm theo từ khóa |
| Hallucination | Mô hình sinh thông tin sai nhưng nghe hợp lý |
| Faithfulness | Câu trả lời có nằm trong ngữ cảnh đã cung cấp không |
| Knowledge graph | Đồ thị gồm thực thể và quan hệ |
| Community (GraphRAG) | Nhóm thực thể liên quan chặt chẽ, có tóm tắt riêng |
| Agent | Hệ thống LLM kết hợp công cụ và vòng lặp quyết định |
| Tool calling | Mô hình chọn hàm và tham số để gọi công cụ bên ngoài |
| Indirect prompt injection | Chỉ dẫn độc hại ẩn trong dữ liệu được truy hồi |

## Phần 12. Nguồn tham khảo

### 12.1. Đã đối chiếu (đọc phần tóm tắt hoặc tóm tắt từ kết quả tìm kiếm)
- Lewis, P. và cộng sự (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. [arXiv:2005.11401](https://arxiv.org/abs/2005.11401).
- Edge, D. và cộng sự (2024). From Local to Global: A Graph RAG Approach to Query-Focused Summarization. [arXiv:2404.16130](https://arxiv.org/abs/2404.16130).
- Singh, A. và cộng sự. Agentic Retrieval-Augmented Generation: A Survey on Agentic RAG. [arXiv:2501.09136](https://arxiv.org/pdf/2501.09136). Chỉ tóm tắt, chưa đọc toàn văn.

### 12.2. Chưa đối chiếu trực tiếp
- Tài liệu chính thức của Microsoft GraphRAG (microsoft.github.io/graphrag hoặc kho mã nguồn chính thức). Kết quả tìm kiếm chỉ trả về bản sao và bài hướng dẫn bên thứ ba. Các lệnh CLI và API cần kiểm tra lại với phiên bản đang dùng.
- Các phương pháp Self-RAG, CRAG, HyDE, PlanRAG: tên và ý tưởng dựa trên kiến thức nền, chưa mở bài báo gốc.
- Chỉ số đánh giá (Recall@k, MRR, nDCG, faithfulness): các định nghĩa là phổ biến trong tài liệu tìm kiếm thông tin và đánh giá RAG, nhưng cách tính chi tiết có thể khác giữa thư viện.

### 12.3. Điểm cần xác minh thêm
- Số liệu cải thiện của GraphRAG trên các bộ dữ liệu cụ thể.
- Chi tiết thuật toán phát hiện cộng đồng và tham số mặc định trong thư viện hiện hành.
- Các mẫu prompt chống tiêm lệnh gián tiếp: hiệu quả thực tế cần kiểm thử riêng.

Hết tài liệu RAG, GraphRAG và Agentic RAG.
