# REA (Resources, Events, Agents): Tài liệu tham khảo

> **Cách hiểu "REA" trong tài liệu này:** mô hình kế toán **Resources, Events, Agents** (Tài nguyên, Sự kiện, Tác nhân), do William E. McCarthy đề xuất năm 1982. Đây là cách hiểu phổ biến nhất trong kế toán và hệ thống thông tin. Nếu bạn muốn một "REA" khác (ví dụ một sản phẩm, hay một viết tắt trong lĩnh vực khác), hãy cho tôi biết.
> **Viết cho:** sinh viên kế toán, kỹ sư hệ thống thông tin, người thiết kế ERP hoặc cơ sở dữ liệu kế toán.
> **Ngày tra cứu:** 2026-10-21.
> **Cách đọc nhãn:**
> - `[Đã kiểm chứng]`: đã đối chiếu với nguồn trong lần tra cứu này.
> - `[Trí nhớ]`: kiến thức nền, chưa đối chiếu với bản gốc. Cần kiểm tra trước khi trích dẫn chính thức.
> - `[Suy luận]`: tôi suy ra hoặc minh họa, không phải nội dung trực tiếp từ nguồn.

---

## Mục lục

- Phần A. Bối cảnh và động lực
  - 1. Vấn đề của kế toán kép truyền thống
  - 2. McCarthy và ý tưởng năm 1982
- Phần B. Các khái niệm cốt lõi
  - 3. Resources (Tài nguyên)
  - 4. Events (Sự kiện)
  - 5. Agents (Tác nhân)
  - 6. Duality (Tính đối ngẫu)
- Phần C. Mô hình và quan hệ
  - 7. Mô hình REA cơ bản
  - 8. Các loại quan hệ
  - 9. Quy trình, cam kết và chính sách
- Phần D. Ví dụ minh họa
  - 10. Chu trình bán hàng
  - 11. Chu trình mua hàng
  - 12. Sản xuất và chuyển đổi
- Phần E. So sánh với kế toán kép
  - 13. Điểm khác biệt
  - 14. Lợi và hạn chế
- Phần F. Triển khai
  - 15. Cơ sở dữ liệu quan hệ
  - 16. Liên hệ với ERP
- Phần G. Mở rộng: ontology và chuẩn
  - 17. Geerts và McCarthy: REA như ontology doanh nghiệp
  - 18. ISO/IEC 15944-4 và kinh tế mở
  - 19. Phê bình và các hướng tiếp cận khác
- Phần H. Thuật ngữ
- Phần I. Lộ trình học
- Phần J. Nguồn tham khảo
- Phần K. Điểm cần xác minh

---

# PHẦN A. BỐI CẢNH VÀ ĐỘNG LỰC

## 1. Vấn đề của kế toán kép truyền thống

Kế toán kép (double-entry bookkeeping) ghi mỗi giao dịch thành ít nhất hai bút toán, với tổng nợ bằng tổng có. Phương pháp này ra đời trước máy tính, và được thiết kế để kiểm tra đối chiếu bằng tay. Hệ quả:

- Sổ cái (general ledger) chứa các tài khoản trừu tượng (ví dụ "Doanh thu bán hàng", "Phải thu khách hàng") thay vì các đối tượng kinh tế thật.
- Thông tin chi tiết về hàng hóa, khách hàng, số lượng, thời điểm bị nén vào số dư tài khoản.
- Nhiều thông tin cần cho quản trị (ai đã mua gì, bao nhiêu, khi nào) phải được tái tạo từ nhiều nguồn.

Khi máy tính cho phép lưu trữ chi tiết, câu hỏi là: có thể lưu chính các sự kiện kinh tế, và suy ra bút toán khi cần không?

## 2. McCarthy và ý tưởng năm 1982

William E. McCarthy công bố "The REA Accounting Model: A Generalized Framework for Accounting Systems in a Shared Data Environment" trên *The Accounting Review*, tháng 7 năm 1982. [Đã kiểm chứng]

Ý tưởng chính:
- Mô hình hóa **những gì đã thực sự xảy ra trong thế giới kinh tế**, thay vì các tài khoản kế toán.
- Một **cơ sở dữ liệu dùng chung** phục vụ cả kế toán lẫn phi kế toán (sản xuất, bán hàng, nhân sự), thay vì mỗi phòng ban một hệ thống riêng.
- Các bút toán kép, tài khoản, số dư có thể được **sinh ra** từ dữ liệu sự kiện khi cần, không cần lưu trữ trực tiếp như là nguồn chính.

McCarthy mở rộng mô hình thực thể - quan hệ (Entity-Relationship) của Peter Chen (1976). [Đã kiểm chứng]

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 3. Resources (Tài nguyên)

**Định nghĩa:** các đối tượng có giá trị kinh tế mà doanh nghiệp kiểm soát hoặc quan tâm: hàng hóa, dịch vụ, tiền. [Đã kiểm chứng]

Đặc điểm:
- Có thể được **tăng** hoặc **giảm** qua sự kiện.
- Thường được theo dõi theo số lượng, giá trị, vị trí, trạng thái.
- Ví dụ: một lô hàng tồn kho, một tài khoản ngân hàng, một giờ công lao động, một máy móc.

**Lưu ý:** trong REA, tài nguyên là đối tượng thật, không phải số dư. Số dư được tính từ lịch sử tăng giảm.

## 4. Events (Sự kiện)

**Định nghĩa:** các giao dịch hoặc hoạt động kinh tế làm **thay đổi** tài nguyên. [Đã kiểm chứng]

Hai nhóm sự kiện:
- **Sự kiện làm giảm tài nguyên** (give, outflow): bán hàng (giảm hàng tồn), chi tiền (giảm tiền), tiêu hao nguyên liệu.
- **Sự kiện làm tăng tài nguyên** (receive, inflow): mua hàng (tăng hàng tồn), thu tiền (tăng tiền), nhận dịch vụ.

Sự kiện có thời điểm và thường có tham chiếu tới chứng từ gốc (hóa đơn, phiếu nhập kho, phiếu thu).

## 5. Agents (Tác nhân)

**Định nghĩa:** con người hoặc đơn vị tham gia sự kiện: cá nhân, công ty, bộ phận, hoặc hệ thống. [Đã kiểm chứng]

Hai vai trò chính trong một sự kiện:
- **Tác nhân nội bộ:** nhân viên, bộ phận chịu trách nhiệm trong doanh nghiệp.
- **Tác nhân bên ngoài:** khách hàng, nhà cung cấp, ngân hàng, cơ quan thuế.

Mỗi sự kiện có thể có một hoặc nhiều tác nhân tham gia với các vai trò khác nhau (người bán, người trực tiếp thực hiện, người chịu trách nhiệm).

## 6. Duality (Tính đối ngẫu)

**Định nghĩa:** mỗi sự kiện kinh tế có một sự kiện đối ứng, tạo thành cặp trao đổi. Một bên cho đi hoặc mất một tài nguyên, bên kia nhận được hoặc có được tài nguyên. [Đã kiểm chứng]

Ví dụ: trong một giao dịch bán hàng, sự kiện "bán hàng" (giao hàng cho khách, giảm tồn kho) đối ứng với sự kiện "thu tiền" (nhận tiền từ khách, tăng tiền mặt).

Đây là **quan hệ trao đổi** (exchange) trong REA. Không phải mọi sự kiện đều có đối ứng trực tiếp; sản xuất có thể gồm nhiều sự kiện. [Đã kiểm chứng]

**Vì sao quan trọng:** duality đảm bảo rằng mọi thay đổi tài nguyên đều có một lý do kinh tế được ghi nhận, và các cặp này cho phép kiểm tra tính đầy đủ. Đây là vai trò thay thế cho việc kiểm tra cân đối nợ-có bằng tay. [Suy luận]

---

# PHẦN C. MÔ HÌNH VÀ QUAN HỆ

## 7. Mô hình REA cơ bản

Mô hình được vẽ như sau. Đây là phiên bản văn bản, không phải sơ đồ chuẩn:

```
   Agent (Tác nhân A)                              Agent (Tác nhân B)
        │                                                  │
        │ tham gia (participation)                         │ tham gia
        ▼                                                  ▼
  ┌──────────────┐   đối ứng (duality)   ┌──────────────┐
  │ Event        │◄────────────────────►│ Event        │
  │ (giảm/cho đi)│     trao đổi         │ (tăng/nhận)  │
  └──────┬───────┘                      └──────┬───────┘
         │ ảnh hưởng (stockflow)               │ ảnh hưởng (stockflow)
         ▼                                     ▼
  ┌──────────────┐                      ┌──────────────┐
  │ Resource     │                      │ Resource     │
  │ (cho đi)     │                      │ (nhận)       │
  └──────────────┘                      └──────────────┘
```

Trong mô hình thực tế, một sự kiện có thể liên kết với nhiều tài nguyên và nhiều tác nhân.

## 8. Các loại quan hệ

Danh sách dưới đây là các loại quan hệ thường được mô tả trong REA. Tên và cách định nghĩa chi tiết cần đối chiếu với bài gốc của McCarthy (1982). [Trí nhớ]

| Quan hệ | Nối giữa | Ý nghĩa |
|---|---|---|
| **Stockflow** | Sự kiện ↔ Tài nguyên | Sự kiện làm tăng hoặc giảm tài nguyên |
| **Duality** (đối ngẫu) | Sự kiện ↔ Sự kiện | Cặp sự kiện trao đổi lẫn nhau |
| **Participation** (tham gia) | Sự kiện ↔ Tác nhân | Tác nhân tham gia sự kiện, với vai trò cụ thể |
| **Control** (kiểm soát) | Tác nhân ↔ Tài nguyên | Tác nhân có quyền sở hữu hoặc kiểm soát tài nguyên |
| **Responsibility** (trách nhiệm) | Tác nhân ↔ Tác nhân | Một tác nhân chịu trách nhiệm trước tác nhân khác, thường là nhân viên và quản lý |
| **Reciprocal** (đối ứng) | Tác nhân ↔ Tác nhân | Hai bên trong cùng một giao dịch, thường là khách và nhà cung cấp |

**Quan hệ then chốt cho người học:** stockflow và duality. Đây là hai thứ nối dữ liệu với nhau và tạo khả năng suy ra bút toán. [Suy luận]

## 9. Quy trình, cam kết và chính sách

Mô hình gốc mở rộng bằng ba khái niệm quan trọng:

- **Quy trình (business process):** mỗi quy trình như bán hàng, mua hàng, sản xuất có một mô hình REA riêng. [Đã kiểm chứng]
- **Cam kết (commitment):** một sự kiện tương lai chưa xảy ra, ví dụ đơn đặt hàng, hợp đồng. Cam kết được lưu để theo dõi việc thực hiện và công nợ phát sinh. [Đã kiểm chứng: nguồn có nhắc đến mở rộng sang cam kết]
- **Chính sách (policy):** quy tắc kinh doanh quyết định sự kiện nào được phép, ví dụ hạn mức tín dụng. [Đã kiểm chứng: nguồn có nhắc đến mở rộng sang chính sách]

Ba khái niệm này cho phép REA chứa thông tin quản trị ngoài sổ sách: đơn chưa giao, hạn mức, hợp đồng dài hạn.

---

# PHẦN D. VÍ DỤ MINH HỌA

## 10. Chu trình bán hàng

**Tình huống:** Công ty A bán 10 cái ghế cho khách B với giá 1.000.000 đồng mỗi cái, khách trả tiền sau 2 tuần.

| Thành phần | Nội dung |
|---|---|
| Tài nguyên | Hàng tồn kho "ghế" (giảm 10); Tiền mặt/phải thu (tăng 10.000.000) |
| Sự kiện 1 | **Bán hàng**: giao 10 ghế, giảm tồn kho. Tác nhân: A (bên bán), B (bên mua) |
| Sự kiện 2 | **Thu tiền**: nhận 10.000.000 từ B. Tác nhân: B (bên trả), A (bên nhận) |
| Duality | Sự kiện 1 ↔ Sự kiện 2 (trao đổi hàng lấy tiền) |
| Cam kết | Đơn đặt hàng và điều khoản thanh toán 14 ngày, tạo cam kết "thu tiền" chưa thực hiện |
| Chính sách | Hạn mức tín dụng của B được kiểm tra trước khi giao |

**Suy ra bút toán (nếu cần báo cáo tài chính):** các bút toán "Nợ phải thu / Có doanh thu" và "Nợ tiền / Có phải thu" có thể được sinh từ hai sự kiện trên, theo quy tắc ánh xạ. Đây là điểm mấu chốt: bút toán là kết quả, không phải dữ liệu gốc. [Suy luận]

**Liên hệ thực tế:** phần chi phí giá vốn (giảm tồn kho theo giá vốn) cũng có thể được sinh từ sự kiện bán hàng, nếu hệ thống theo dõi giá vốn của từng lô. [Suy luận]

## 11. Chu trình mua hàng

Tương tự chu trình bán hàng, nhưng đảo chiều:

| Thành phần | Nội dung |
|---|---|
| Tài nguyên | Tồn kho tăng; Tiền giảm hoặc phải trả tăng |
| Sự kiện 1 | **Nhận hàng**: nhận nguyên liệu từ nhà cung cấp |
| Sự kiện 2 | **Thanh toán**: chi tiền cho nhà cung cấp |
| Duality | Nhận hàng ↔ Thanh toán |
| Cam kết | Đơn mua, lịch giao hàng, điều khoản thanh toán |

## 12. Sản xuất và chuyển đổi

Sản xuất không phải là trao đổi hai bên, mà là **chuyển đổi** nguyên liệu thành thành phẩm. Mô hình REA xử lý bằng nhiều sự kiện:

```
Nguyên liệu (giảm) ──► Sự kiện "tiêu hao nguyên liệu"  ─┐
Lao động (giảm)    ──► Sự kiện "ghi nhận công lao động" ─┤──► Sự kiện "nhập thành phẩm" (tăng)
Máy móc (hao mòn)  ──► Sự kiện "khấu hao"                ─┘
```

Đây là ví dụ về việc duality không đơn giản là cặp hai sự kiện. Nhiều sự kiện có thể cùng liên kết với một sự kiện tổng hợp. [Đã kiểm chứng: nguồn nêu rằng sản xuất có thể có nhiều hơn hai sự kiện]

---

# PHẦN E. SO SÁNH VỚI KẾ TOÁN KÉP

## 13. Điểm khác biệt

| Tiêu chí | Kế toán kép | REA |
|---|---|---|
| Đơn vị lưu trữ | Bút toán, tài khoản, số dư | Sự kiện, tài nguyên, tác nhân |
| Kiểm tra tính đúng | Tổng nợ = tổng có | Duality và ràng buộc quan hệ |
| Chi tiết | Nén theo tài khoản | Giữ chi tiết từng sự kiện |
| Nguồn gốc dữ liệu | Bút toán là gốc | Sự kiện là gốc, bút toán là suy ra |
| Tính phi tài chính | Hạn chế | Tự nhiên: số lượng, vị trí, người phụ trách |
| Đa mục đích | Chủ yếu báo cáo tài chính | Kế toán, quản trị, vận hành dùng chung một dữ liệu |
| Dễ học | Có truyền thống lâu đời | Cần hiểu mô hình dữ liệu |

**Lưu ý:** REA không thay thế kế toán kép trong báo cáo pháp định. Nó đóng vai trò lớp dữ liệu gốc; báo cáo vẫn cần ánh xạ sang tài khoản theo chuẩn kế toán. [Suy luận]

## 14. Lợi và hạn chế

**Lợi thế:**
- Một nguồn dữ liệu dùng chung cho kế toán, bán hàng, kho, sản xuất.
- Truy vấn linh hoạt theo số lượng, tác nhân, thời điểm, không bị giới hạn bởi tài khoản.
- Dễ mở rộng cho cam kết, chính sách, quản trị rủi ro.
- Giảm trùng lặp dữ liệu giữa các hệ thống.

**Hạn chế:**
- Đòi hỏi thiết kế dữ liệu cẩn thận; mô hình chưa quen với nhiều kế toán viên.
- Ánh xạ sang báo cáo tài chính chuẩn cần quy tắc rõ ràng.
- Một số nguồn cho rằng mô hình khó biểu diễn chính xác các quan hệ trong đồ thị đồ họa. [Đã kiểm chứng: có phê bình về tính mơ hồ của quan hệ, xem mục 19]
- Thay đổi từ hệ thống cũ sang REA có chi phí chuyển đổi đáng kể.

---

# PHẦN F. TRIỂN KHAI

## 15. Cơ sở dữ liệu quan hệ

Nguồn cho biết hệ thống REA thường được mô hình hóa như cơ sở dữ liệu quan hệ với sơ đồ thực thể - quan hệ, nhưng không bắt buộc. [Đã kiểm chứng]

Bảng minh họa (đơn giản hóa, không phải thiết kế hoàn chỉnh):

```sql
-- Tác nhân
CREATE TABLE agent (
  agent_id     INTEGER PRIMARY KEY,
  name         TEXT NOT NULL,
  agent_type   TEXT CHECK (agent_type IN ('internal', 'external'))
);

-- Tài nguyên
CREATE TABLE resource (
  resource_id  INTEGER PRIMARY KEY,
  name         TEXT NOT NULL,
  unit         TEXT,
  resource_type TEXT   -- ví dụ: 'inventory', 'cash', 'receivable'
);

-- Sự kiện
CREATE TABLE event (
  event_id     INTEGER PRIMARY KEY,
  event_type   TEXT NOT NULL,   -- 'sale', 'cash_receipt', 'purchase', ...
  occurred_at  TIMESTAMP NOT NULL
);

-- Quan hệ stockflow: sự kiện tác động lên tài nguyên
CREATE TABLE stockflow (
  event_id     INTEGER REFERENCES event(event_id),
  resource_id  INTEGER REFERENCES resource(resource_id),
  quantity     NUMERIC NOT NULL,   -- dương: tăng; âm: giảm
  value        NUMERIC,
  PRIMARY KEY (event_id, resource_id)
);

-- Quan hệ duality: cặp sự kiện trao đổi
CREATE TABLE duality (
  event_give_id    INTEGER REFERENCES event(event_id),
  event_receive_id INTEGER REFERENCES event(event_id),
  PRIMARY KEY (event_give_id, event_receive_id)
);

-- Quan hệ participation: tác nhân tham gia sự kiện
CREATE TABLE participation (
  event_id     INTEGER REFERENCES event(event_id),
  agent_id     INTEGER REFERENCES agent(agent_id),
  role         TEXT NOT NULL,   -- 'seller', 'buyer', 'responsible', ...
  PRIMARY KEY (event_id, agent_id, role)
);
```

**Ghi chú thiết kế:**
- Ràng buộc duality (mọi sự kiện "cho đi" có đối ứng) thường được kiểm tra ở tầng ứng dụng hoặc bằng trigger, vì SQL không biểu diễn trực tiếp. [Suy luận]
- Số dư tồn kho hoặc tiền là **truy vấn** tổng hợp trên `stockflow`, không lưu trực tiếp. [Suy luận]

## 16. Liên hệ với ERP

Một bài báo nghiên cứu (Fallon và Polovina, 2013) cho thấy REA có thể dùng để mô hình hóa quy trình ERP hiện có, cụ thể là SAP HCM. [Đã kiểm chứng: nguồn có tiêu đề bài báo, chưa đọc nội dung chi tiết]

Trong thực tế, REA thường được dùng để:
- Thiết kế mô hình dữ liệu khi xây dựng ERP;
- Phân tích quy trình hiện có;
- Đối chiếu các phân hệ (bán hàng, kho, kế toán) xem có trùng lặp dữ liệu không.

---

# PHẦN G. MỞ RỘNG: ONTOLOGY VÀ CHUẨN

## 17. Geerts và McCarthy: REA như ontology doanh nghiệp

- **Năm 2002:** Guido Geerts và William McCarthy trình bày REA như một ontology miền kinh doanh, nằm trong ontology cấp trên của Sowa. Mở rộng gồm cả hiện tượng nội bộ (ERP) và liên doanh nghiệp (chuỗi cung ứng). [Đã kiểm chứng]
- Bài phân tích chính: "An ontological analysis of the economic primitives of the extended-REA enterprise information architecture" (2002). [Đã kiểm chứng: tiêu đề có trong nguồn]
- Geerts và McCarthy cũng có một working paper năm 2000 về nền tảng ontology của REA. [Đã kiểm chứng: tiêu đề có trong nguồn]
- Có một monograph năm 2022 của McCarthy cùng Geerts và Gal, có bản nháp trên trang của McCarthy. [Đã kiểm chứng]

## 18. ISO/IEC 15944-4 và kinh tế mở

- REA ảnh hưởng đến chuẩn **ISO/IEC 15944-4**, công bố tháng 11 năm 2007, về ontology kế toán và kinh tế cho kinh tế mở (Open-edi), phần liên doanh nghiệp. [Đã kiểm chứng]
- REA cũng ảnh hưởng đến ebXML, theo Wikipedia, với McCarthy tham gia ủy ban chuẩn. [Đã kiểm chứng: nguồn Wikipedia; cần đối chiếu với nguồn chính thức]
- The Open Group công nhận REA trong TOGAF như một công cụ mô hình hóa quy trình nghiệp vụ hữu ích. [Đã kiểm chứng: nguồn Wikipedia; cần đối chiếu]

## 19. Phê bình và các hướng tiếp cận khác

**Phê bình được ghi nhận:**
- Ontology REA đôi khi mơ hồ trong định nghĩa quan hệ giữa các khái niệm cốt lõi; thiếu ngôn ngữ mô hình hóa chính xác và ký hiệu đồ họa dễ dùng. [Đã kiểm chứng]
- Một số nguồn cho rằng REA không cần kế toán kép; tuy nhiên đây là cách nhìn quảng bá, cần thận trọng. [Đã kiểm chứng: có nhận định này trong nguồn]
- Các nhà phê bình đề xuất dùng OntoUML (ngôn ngữ do Guizzardi phát triển) để tái cấu trúc ontology REA theo mặt siêu hình học. Gailly và cộng sự đã áp dụng. [Đã kiểm chứng]
- Laurier, Bernaert và Poels đề xuất một mô hình tham chiếu thống nhất, biểu diễn được trao đổi, chuyển giao và chuyển đổi, mà mô hình McCarthy và Hruby không biểu diễn đầy đủ. [Đã kiểm chứng]

**Các hướng tiếp cận khác đáng so sánh:**
- **XBRL GL:** chuẩn trao đổi báo cáo kế toán, được mô tả là bám sát kế toán kép. Hai hướng bị đối lập trong nguồn. [Đã kiểm chứng]
- **Mô hình Hruby (Hruby, Kiehn, Scheller 2006):** mô hình mẫu doanh nghiệp (business patterns). [Đã kiểm chứng: tiêu đề có trong nguồn]

---

# PHẦN H. THUẬT NGỮ

| Thuật ngữ (Anh) | Tiếng Việt | Giải thích ngắn |
|---|---|---|
| Resource | Tài nguyên | Đối tượng có giá trị kinh tế |
| Event | Sự kiện | Giao dịch làm thay đổi tài nguyên |
| Agent | Tác nhân | Người hoặc đơn vị tham gia sự kiện |
| Duality | Đối ngẫu | Cặp sự kiện trao đổi lẫn nhau |
| Stockflow | Dòng tài nguyên | Sự kiện tăng hoặc giảm tài nguyên |
| Participation | Tham gia | Tác nhân tham gia sự kiện, có vai trò |
| Control | Kiểm soát | Tác nhân kiểm soát tài nguyên |
| Responsibility | Trách nhiệm | Quan hệ giữa tác nhân và tác nhân |
| Reciprocal | Đối ứng | Quan hệ hai bên trong giao dịch |
| Commitment | Cam kết | Sự kiện tương lai đã được thỏa thuận |
| Policy | Chính sách | Quy tắc quyết định sự kiện nào được phép |
| Ontology | Ontology (bản thể học dữ liệu) | Mô tả chính thức các khái niệm và quan hệ trong một lĩnh vực |
| Double-entry bookkeeping | Kế toán kép | Ghi mỗi giao dịch thành nợ và có |
| Entity-Relationship | Thực thể - quan hệ | Mô hình dữ liệu của Chen (1976) |
| Business process | Quy trình nghiệp vụ | Chuỗi hoạt động tạo giá trị |
| ERP | Hoạch định nguồn lực doanh nghiệp | Hệ thống tích hợp các phân hệ |
| Open-edi | Trao đổi điện tử mở | Khung kinh tế điện tử theo ISO |

---

# PHẦN I. LỘ TRÌNH HỌC

1. **Nắm kế toán kép:** tài khoản, bút toán, nợ-có, báo cáo tài chính cơ bản. Đây là nền để thấy REA khác ở đâu.
2. **Đọc bài gốc của McCarthy (1982)** trên *The Accounting Review*. Bài ngắn, có thể đọc trong vài giờ.
3. **Tự vẽ mô hình REA** cho một chu trình quen thuộc (ví dụ mua cà phê từ nông dân, bán cho quán).
4. **Viết truy vấn** trên mô hình SQL ở mục 15 để tính tồn kho và công nợ từ sự kiện.
5. **Đọc Geerts và McCarthy (2002)** để hiểu mở rộng sang ontology.
6. **Tìm hiểu phê bình** (Laurier, Bernaert, Poels) để có cái nhìn cân bằng.

---

# PHẦN J. NGUỒN THAM KHẢO

**Bài gốc và nền tảng:**
- McCarthy, W. E. (1982). "The REA Accounting Model: A Generalized Framework for Accounting Systems in a Shared Data Environment." *The Accounting Review*, July 1982, pp. 554–578. Có trên ResearchGate: https://www.researchgate.net/publication/239064746_The_REA_Accounting_Model_A_Generalized_Framework_for_Accounting_Systems_Shared_Data_Environment [đã tra cứu, mục tiêu tiêu đề và thông tin xuất bản]
- Chen, P. P. (1976). Mô hình thực thể - quan hệ. [Trí nhớ]

**Tổng quan và lịch sử:**
- Wikipedia, "Resources, Events, Agents": https://en.wikipedia.org/wiki/Resources,_Events,_Agents [đã tra cứu]
- HandWiki, "Finance:Resources, Events, Agents": https://handwiki.org/wiki/Finance:Resources,_Events,_Agents [bản sao của Wikipedia, không độc lập]
- P2P Foundation, "Resource-Event-Agent Model": https://wiki.p2pfoundation.net/Resource-Event-Agent_Model [có nhận định chưa được xác nhận, cần thận trọng]
- McCarthy, trang cá nhân tại MSU: https://msu.edu/~mccarth4/chen-con.html [đã tra cứu, trang liên quan đến monograph năm 2022]

**Ontology và mở rộng:**
- Geerts, G. và McCarthy, W. E. (2002). "An Ontological Analysis of the Economic Primitives of the Extended-REA Enterprise Information Architecture." [đã tra cứu tiêu đề; cần tìm bản đầy đủ]
- Geerts và McCarthy (2000). Working paper về nền tảng ontology. [đã tra cứu tiêu đề]
- Fallon và Polovina (2013). "REA Analysis of SAP HCM; Some Initial Findings." [đã tra cứu tiêu đề]
- Partridge (2002). "Steps towards the development of a reference ontology for accounting." [đã tra cứu tiêu đề]
- Hruby, Kiehn, Scheller (2006). *Model-Driven Design Using Business Patterns*. [đã tra cứu tiêu đề]
- Dunn, Cherrington, Hollander (2004). *Enterprise Information Systems: A Pattern-Based Approach*. [đã tra cứu tiêu đề]
- Hollander, Denna, Cherrington (1999). *Accounting, Information Technology, and Business Solutions*. [đã tra cứu tiêu đề]

**Chuẩn:**
- ISO/IEC 15944-4 (Open-edi, ontology kế toán và kinh tế). [đã tra cứu tên chuẩn; cần truy cập trang ISO để xác nhận]
- ebXML. [đã tra cứu tên chuẩn]
- TOGAF (The Open Group). [đã tra cứu tên chuẩn]

**Phê bình và so sánh:**
- Laurier, Bernaert, Poels: mô hình tham chiếu thống nhất. [đã tra cứu tên tác giả, cần tìm bài đầy đủ]
- Guizzardi, OntoUML. [Trí nhớ]
- Gailly và cộng sự, tái cấu trúc REA theo OntoUML. [đã tra cứu tên tác giả, cần tìm bài đầy đủ]
- Fischer-Pauzenberger và Schwaiger. [đã tra cứu tên tác giả, cần tìm bài đầy đủ]
- Đường dẫn bài tổng hợp về REA và ontology (online-cig.ase.ro, ideas.repec.org, scitepress.org): có thể đọc để so sánh thêm. [đã tra cứu, nội dung chưa đọc chi tiết]

---

# PHẦN K. ĐIỂM CẦN XÁC MINH

1. **Danh sách và định nghĩa chính xác các quan hệ** (stockflow, duality, participation, control, responsibility, reciprocal) cần đối chiếu với bài năm 1982. Mục 8 dựa vào kiến thức nền.
2. **Ràng buộc duality** cần tìm nguồn chi tiết; tìm kiếm không cho ra định nghĩa đầy đủ.
3. **Sơ đồ REA cơ bản** trên Wikipedia là hình ảnh; tôi chỉ mô tả bằng văn bản ở mục 7.
4. **Các phê bình** về mức độ "không cần kế toán kép" là nhận định cần kiểm chứng từ nghiên cứu học thuật.
5. **Nội dung chi tiết** của các bài Fallon–Polovina, Geerts–McCarthy 2002, Laurier–Bernaert–Poels chưa được đọc trực tiếp.
6. **Chuẩn ISO/IEC 15944-4** và ebXML: cần đối chiếu với trang chính thức.

---

**Tóm lược:**
- REA = Tài nguyên, Sự kiện, Tác nhân. Mô hình dữ liệu kinh tế thay cho kế toán theo tài khoản.
- Trục chính là **sự kiện** và **đối ngẫu trao đổi**; bút toán kép được **suy ra** khi cần.
- Mở rộng thành **ontology doanh nghiệp** và chuẩn ISO 15944-4.
- Có phê bình về tính mơ hồ của quan hệ, nên học kèm với các hướng tiếp cận như OntoUML.
