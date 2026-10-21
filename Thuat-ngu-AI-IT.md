# Từ điển thuật ngữ AI và IT

Tài liệu tổng hợp các thuật ngữ thường gặp trong AI và IT, phân theo chủ đề. Mỗi thuật ngữ có:

- **Phiên âm:** cách đọc gần đúng bằng chữ Việt, để người đọc tiếng Việt đọc được. Đây không phải phiên âm IPA chuẩn.
- **Nghĩa:** nghĩa tiếng Việt ngắn gọn.
- **Giải thích:** mô tả thêm để hiểu vai trò hoặc ngữ cảnh dùng.

Ghi chú: các định nghĩa dựa trên cách dùng phổ biến trong ngành tính đến thời điểm viết. Một số thuật ngữ AI mới (đặc biệt về agent và giao thức) đang thay đổi nhanh, nên nên đối chiếu với tài liệu chính thức khi cần chính xác tuyệt đối.

Mục lục:
- [Phần 1. Nền tảng máy tính và hệ điều hành](#phần-1-nền-tảng-máy-tính-và-hệ-điều-hành)
- [Phần 2. Mạng và Internet](#phần-2-mạng-và-internet)
- [Phần 3. Lập trình và phần mềm](#phần-3-lập-trình-và-phần-mềm)
- [Phần 4. Cơ sở dữ liệu](#phần-4-cơ-sở-dữ-liệu)
- [Phần 5. DevOps, cloud và container](#phần-5-devops-cloud-và-container)
- [Phần 6. Bảo mật thông tin](#phần-6-bảo-mật-thông-tin)
- [Phần 7. Dữ liệu và thống kê](#phần-7-dữ-liệu-và-thống-kê)
- [Phần 8. Trí tuệ nhân tạo và học máy](#phần-8-trí-tuệ-nhân-tạo-và-học-máy)
- [Phần 9. Mô hình ngôn ngữ lớn và AI tạo sinh](#phần-9-mô-hình-ngôn-ngữ-lớn-và-ai-tạo-sinh)
- [Phần 10. Agent, RAG và giao thức tích hợp](#phần-10-agent-rag-và-giao-thức-tích-hợp)
- [Phần 11. Vận hành và đánh giá AI](#phần-11-vận-hành-và-đánh-giá-ai)

---

## Phần 1. Nền tảng máy tính và hệ điều hành

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| CPU | xi-pi-iu | Bộ xử lý trung tâm | Thành phần thực hiện các phép tính và lệnh của chương trình |
| GPU | gi-pi-iu | Bộ xử lý đồ họa | Có nhiều nhân nhỏ, phù hợp tính toán song song như huấn luyện và chạy mô hình AI |
| RAM | ram | Bộ nhớ truy cập ngẫu nhiên | Bộ nhớ tạm, mất dữ liệu khi tắt máy. Chương trình đang chạy nằm ở đây |
| ROM | rôm | Bộ nhớ chỉ đọc | Bộ nhớ lưu cố định, thường dùng cho firmware |
| Ổ cứng SSD | ét-ét-đi | Ổ thể rắn | Lưu trữ không có bộ phận cơ học, nhanh hơn HDD |
| HDD | hát-đi-đi | Ổ cứng đĩa từ | Lưu trữ bằng đĩa từ quay, dung lượng lớn nhưng chậm hơn SSD |
| Cache | kết-s | Bộ nhớ đệm | Vùng lưu tạm dữ liệu hay dùng để truy cập nhanh hơn |
| Firmware | phơm-uê | Phần mềm nhúng | Chương trình cấp thấp gắn với phần cứng, ví dụ BIOS/UEFI |
| BIOS / UEFI | bai-ốt / diu-ê-phai | Phần mềm khởi động | Khởi tạo phần cứng trước khi hệ điều hành chạy |
| Hệ điều hành (OS) | ô-ét | Operating System | Phần mềm quản lý phần cứng và cung cấp môi trường cho ứng dụng |
| Tiến trình (process) | prâu-sét | Tiến trình | Một chương trình đang chạy, có không gian bộ nhớ riêng |
| Luồng (thread) | rét | Luồng | Đơn vị thực thi nhỏ hơn tiến trình, các luồng chung bộ nhớ |
| Kernel | cơ-nồ | Nhân hệ điều hành | Phần lõi quản lý tài nguyên, tiến trình và bộ nhớ |
| Ảo hóa (virtualization) | vơ-chu-ồ-li-di-sần | Ảo hóa | Chạy nhiều hệ thống trên cùng một máy vật lý |
| Máy ảo (VM) | vi-em | Virtual Machine | Máy tính giả lập chạy trên hypervisor, có hệ điều hành riêng |
| Hypervisor | hai-pơ-vai-dơ | Trình quản lý máy ảo | Phần mềm tạo và chạy máy ảo |
| Dual boot | điu-ồ bút | Khởi động kép | Cài hai hệ điều hành trên một máy và chọn khi khởi động |
| Driver | đrai-vơ | Trình điều khiển | Phần mềm để hệ điều hành giao tiếp với một thiết bị phần cứng |
| BSOD | bi-ét-ô-đi | Màn hình xanh chết chóc | Màn hình lỗi nghiêm trọng của Windows khi hệ thống gặp sự cố không phục hồi |
| Shell | sen | Vỏ dòng lệnh | Giao diện để người dùng nhập lệnh cho hệ điều hành, ví dụ CMD, PowerShell, Bash |
| Bash | bét | Shell Unix | Shell phổ biến trên Linux và macOS |
| Terminal | tơ-mi-nồ | Cửa sổ dòng lệnh | Ứng dụng hiển thị và nhận lệnh từ shell |
| Linux | lí-nắc | Hệ điều hành mã nguồn mở | Dựa trên kernel Linux, có nhiều bản phân phối |
| Distro | đi-xtrô | Bản phân phối | Bản Linux đóng gói kernel cùng phần mềm, ví dụ Ubuntu, Debian |

---

## Phần 2. Mạng và Internet

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| IP | ai-pi | Giao thức Internet | Quy tắc định địa chỉ và chuyển gói tin giữa các máy |
| IP address | ai-pi ét-rét-x | Địa chỉ IP | Số định danh máy trên mạng, ví dụ 192.168.1.10 |
| IPv4 | ai-pi vi-bốn | Internet Protocol phiên bản 4 | Địa chỉ 32 bit, đã cạn dần |
| IPv6 | ai-pi vi-sáu | Internet Protocol phiên bản 6 | Địa chỉ 128 bit, có không gian địa chỉ lớn hơn nhiều |
| TCP | ti-si-pi | Giao thức điều khiển truyền | Truyền dữ liệu có kết nối, đảm bảo đến đúng thứ tự |
| UDP | iu-đi-pi | Giao thức gói dữ liệu người dùng | Truyền không kết nối, nhanh hơn, không đảm bảo đến đủ |
| HTTP | hát-ti-ti-pi | Giao thức truyền siêu văn bản | Nền tảng của giao tiếp web |
| HTTPS | hát-ti-ti-pi-ét | HTTP có mã hóa TLS | Phiên bản HTTP bảo mật |
| DNS | đi-en-ét | Hệ thống tên miền | Chuyển tên miền như example.com thành địa chỉ IP |
| DHCP | đi-hát-xê-pi | Giao thức cấp địa chỉ động | Tự động cấp IP, gateway, DNS cho thiết bị |
| Gateway | ghết-uây | Cổng ra | Thiết bị chuyển gói tin ra mạng ngoài |
| Router | rau-tơ | Bộ định tuyến | Thiết bị chọn đường đi cho gói tin giữa các mạng |
| Switch | xuýt | Bộ chuyển mạch | Thiết bị nối các máy trong cùng mạng nội bộ |
| Firewall | pha-ơ-oan | Tường lửa | Lọc lưu lượng theo quy tắc để chặn truy cập không mong muốn |
| NAT | en-ây-ti | Dịch địa chỉ mạng | Đổi địa chỉ IP nội bộ sang IP công cộng khi ra Internet |
| VPN | vi-pi-en | Mạng riêng ảo | Tạo đường hầm mã hóa qua mạng công cộng |
| Port | pót | Cổng mạng | Số định danh dịch vụ trên một máy, ví dụ 443 cho HTTPS |
| Socket | xốc-kít | Ổ cắm mạng | Điểm cuối kết nối gồm IP và cổng |
| Latency | lây-ten-xi | Độ trễ | Thời gian để gói tin đi và về |
| Bandwidth | béng-uít | Băng thông | Lượng dữ liệu tối đa truyền được trong một đơn vị thời gian |
| Packet | pa-kít | Gói tin | Đơn vị dữ liệu nhỏ được truyền trên mạng |
| CDN | xi-đi-en | Mạng phân phối nội dung | Mạng máy chủ đặt gần người dùng để phục vụ nội dung nhanh |
| Load balancer | lốt ben-xơ | Bộ cân bằng tải | Chia yêu cầu đến nhiều máy chủ |
| REST | rét | Representational State Transfer | Kiểu thiết kế API dùng HTTP và tài nguyên |
| WebSocket | uép-xóc-kít | Kênh hai chiều qua web | Kết nối liên tục cho phép server và client gửi dữ liệu bất kỳ lúc nào |
| gRPC | gi-rếp-si | gRPC Remote Procedure Call | Framework gọi hàm từ xa hiệu năng cao, dùng Protocol Buffers |

---

## Phần 3. Lập trình và phần mềm

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| Source code | xo-xơ cốt | Mã nguồn | Mã viết bằng ngôn ngữ lập trình |
| Biên dịch (compile) | com-pai | Biên dịch | Chuyển mã nguồn sang mã máy hoặc dạng trung gian |
| Thông dịch (interpret) | in-tơ-pờ-rét | Thông dịch | Thực thi mã từng dòng khi chạy |
| Biến (variable) | ve-ri-ơ-bồ | Biến | Vùng nhớ có tên để lưu giá trị |
| Hằng số (constant) | cơn-x-tần | Hằng | Giá trị không thay đổi sau khi gán |
| Hàm (function) | phăng-sần | Hàm | Khối lệnh có tên, nhận đầu vào và trả về kết quả |
| Tham số (parameter) | pa-ram-i-tơ | Tham số | Biến nhận giá trị khi gọi hàm |
| Đối số (argument) | a-gơ-mần | Đối số | Giá trị truyền vào khi gọi hàm |
| Đệ quy (recursion) | rì-cơ-sần | Đệ quy | Hàm tự gọi chính nó với bài toán nhỏ hơn |
| Kiểu dữ liệu (data type) | đây-tơ tai-típ | Kiểu dữ liệu | Quy định loại giá trị và thao tác được phép |
| Ép kiểu (casting) | cát-tinh | Chuyển kiểu | Chuyển giá trị từ kiểu này sang kiểu khác |
| Lớp (class) | cờ-lát | Lớp | Khuôn mẫu tạo đối tượng, gồm thuộc tính và phương thức |
| Đối tượng (object) | óp-dêch | Đối tượng | Thể hiện cụ thể của một lớp |
| Đóng gói (encapsulation) | en-cáp-xu-lây-sần | Đóng gói | Giấu chi tiết bên trong, chỉ lộ giao diện cần thiết |
| Kế thừa (inheritance) | in-héc-i-tần | Kế thừa | Lớp con nhận thuộc tính và phương thức từ lớp cha |
| Đa hình (polymorphism) | pô-li-mo-phi-dắm | Đa hình | Cùng một giao diện nhưng hành vi khác nhau tùy đối tượng |
| Giao diện (interface) | in-tơ-phây | Giao diện lập trình | Tập hợp hợp đồng phương thức mà lớp phải cung cấp |
| Thư viện (library) | lai-brê-ri | Thư viện | Tập mã tái sử dụng, ứng dụng gọi vào |
| Framework | phrêm-uơ | Khung phát triển | Bộ khung có sẵn cấu trúc, ứng dụng được xây dựng theo quy ước của nó |
| Module | mô-diu | Mô-đun | Đơn vị mã độc lập, có thể tái sử dụng |
| API | ây-pi-ai | Application Programming Interface | Giao diện để các phần mềm giao tiếp với nhau |
| SDK | ét-đi-kê | Bộ phát triển phần mềm | Gói công cụ, thư viện và tài liệu để phát triển cho một nền tảng |
| IDE | ai-đi-i | Môi trường phát triển tích hợp | Ứng dụng gom soạn mã, gỡ lỗi, build vào một giao diện, ví dụ VS Code |
| Debug | đi-băng | Gỡ lỗi | Tìm và sửa lỗi trong mã |
| Breakpoint | brếch-pôint | Điểm dừng | Vị trí chương trình tạm dừng để kiểm tra khi gỡ lỗi |
| Unit test | diu-nít test | Kiểm thử đơn vị | Kiểm tra một hàm hoặc lớp riêng lẻ |
| Integration test | in-tơ-grết-sần test | Kiểm thử tích hợp | Kiểm tra nhiều thành phần làm việc cùng nhau |
| Regression | ri-grết-sần | Hồi quy | Lỗi xuất hiện lại sau khi sửa đổi mã |
| Refactor | ri-phác-tơ | Tái cấu trúc | Sửa cấu trúc mã mà không đổi hành vi bên ngoài |
| Git | ghít | Hệ thống quản lý phiên bản | Theo dõi lịch sử thay đổi mã |
| Commit | com-mít | Ghi nhận thay đổi | Lưu một ảnh chụp thay đổi vào lịch sử Git |
| Branch | bờ-răng | Nhánh | Dòng phát triển độc lập trong Git |
| Merge | mớt | Hợp nhất | Gộp thay đổi từ nhánh này vào nhánh khác |
| Pull request | pun-rếch-ét | Yêu cầu hợp nhất | Đề nghị gộp nhánh, kèm đánh giá mã |
| Open source | âu-pần xo-xơ | Mã nguồn mở | Mã công khai, người khác có thể xem, sửa và phân phối theo giấy phép |
| Giấy phép (license) | lai-xần | Giấy phép | Điều khoản quy định cách dùng phần mềm |
| Dependency | đi-pen-đen-xi | Phụ thuộc | Thư viện mà dự án cần để chạy |
| Package manager | pác-kít man-nờ-giơ | Trình quản lý gói | Công cụ cài đặt, cập nhật thư viện, ví dụ npm, pip |
| Build | bin | Xây dựng sản phẩm | Quá trình biến mã nguồn thành sản phẩm chạy được |
| Runtime | răn-tai-mơ | Môi trường thực thi | Môi trường chạy chương trình, ví dụ JVM, Node.js |
| Deadlock | đét-lốc | Khóa chết | Các tiến trình chờ nhau mãi không giải phóng tài nguyên |
| Race condition | rết con-đi-sần | Điều kiện đua | Kết quả phụ thuộc thứ tự chạy không kiểm soát của các luồng |
| Memory leak | mem-ơ-ri líc | Rò rỉ bộ nhớ | Bộ nhớ được cấp nhưng không được giải phóng |
| Garbage collection | gác-bếch cô-lếch-sần | Thu gom rác | Tự động giải phóng bộ nhớ không còn được dùng |
| Idempotent | ai-đem-pô-tần | Có tính lũy đẳng | Gọi nhiều lần vẫn cho kết quả như gọi một lần |

---

## Phần 4. Cơ sở dữ liệu

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| Database | đây-tơ-bây-x | Cơ sở dữ liệu | Tập dữ liệu có tổ chức, được quản lý bởi hệ quản trị |
| DBMS | đi-bi-em-ét | Hệ quản trị cơ sở dữ liệu | Phần mềm quản lý dữ liệu, ví dụ MySQL, PostgreSQL |
| RDBMS | a-đi-bi-em-ét | Hệ quản trị CSDL quan hệ | DBMS lưu dữ liệu dạng bảng có quan hệ |
| SQL | ét-kiu-eo | Ngôn ngữ truy vấn có cấu trúc | Ngôn ngữ để truy vấn và thao tác dữ liệu trong CSDL quan hệ |
| Bảng (table) | têy-bồ | Bảng | Cấu trúc lưu dữ liệu theo hàng và cột |
| Khóa chính (primary key) | prai-mơ-ri ki | Khóa chính | Cột hoặc tập cột định danh duy nhất mỗi hàng |
| Khóa ngoại (foreign key) | pho-rần ki | Khóa ngoại | Cột tham chiếu đến khóa chính của bảng khác |
| Index | in-đếch-x | Chỉ mục | Cấu trúc giúp tìm dữ liệu nhanh hơn |
| Transaction | trần-zếch-sần | Giao dịch | Nhóm thao tác được thực hiện trọn vẹn hoặc không làm gì |
| ACID | ây-xít | Atomicity, Consistency, Isolation, Durability | Bộ bốn tính chất đảm bảo giao dịch đáng tin cậy |
| Normalization | nor-mơ-lai-dây-sần | Chuẩn hóa | Tổ chức bảng để giảm trùng lặp dữ liệu |
| Join | giôin | Kết nối bảng | Ghép dữ liệu từ nhiều bảng theo điều kiện |
| View | vi-diu | Khung nhìn | Truy vấn đã lưu, hoạt động như bảng ảo |
| Stored procedure | xtô-tơ prô-xi-dơ | Thủ tục lưu sẵn | Đoạn mã SQL lưu trong CSDL để tái sử dụng |
| Replication | rép-li-kây-sần | Sao chép | Sao dữ liệu sang máy khác để dự phòng hoặc tăng tải đọc |
| Sharding | sác-đinh | Phân mảnh | Chia dữ liệu thành nhiều phần lưu trên nhiều máy |
| NoSQL | nô ét-kiu-eo | Không chỉ SQL | Nhóm CSDL không dùng mô hình bảng quan hệ truyền thống |
| Document store | đác-kiu-mần x-tô | CSDL tài liệu | Lưu dữ liệu dạng tài liệu như JSON, ví dụ MongoDB |
| Key-value store | ki-va-liu x-tô | CSDL khóa-giá trị | Lưu cặp khóa và giá trị, ví dụ Redis |
| Vector database | véc-tơ đây-tơ-bây-x | CSDL vector | Lưu và tìm kiếm vector theo độ tương đồng |
| Data warehouse | đây-tơ oa-hao-x | Kho dữ liệu | Kho lưu dữ liệu lịch sử để phân tích |
| Data lake | đây-tơ lếch | Hồ dữ liệu | Kho lưu dữ liệu thô ở nhiều định dạng |
| ETL | i-ti-eo-eo | Extract, Transform, Load | Quy trình trích xuất, biến đổi và nạp dữ liệu |
| Schema | xki-ma | Lược đồ | Mô tả cấu trúc dữ liệu, các bảng và kiểu |
| Migration | mai-grây-sần | Di chuyển, nâng cấp | Thay đổi cấu trúc CSDL theo phiên bản |
| Backup | béc-áp | Sao lưu | Bản sao dữ liệu để khôi phục khi sự cố |
| Query plan | quê-ri plan | Kế hoạch truy vấn | Cách CSDL thực hiện một truy vấn |

---

## Phần 5. DevOps, cloud và container

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| DevOps | đép-ốp | Phát triển và vận hành | Văn hóa và quy trình kết hợp phát triển và vận hành |
| CI/CD | xi-ai xi-đi | Tích hợp liên tục và triển khai liên tục | Tự động kiểm thử, build và đưa phần mềm lên môi trường |
| Pipeline | pai-plai | Đường ống CI/CD | Chuỗi bước tự động từ commit đến triển khai |
| IaC | ai-ây-xi | Infrastructure as Code | Mô tả hạ tầng bằng mã, ví dụ Terraform |
| Cloud | clao-đ | Đám mây | Hạ tầng và dịch vụ máy tính thuê qua Internet |
| IaaS | ai-ây-ét | Infrastructure as a Service | Thuê hạ tầng ảo như máy chủ và mạng |
| PaaS | pi-ây-ét | Platform as a Service | Thuê nền tảng để chạy ứng dụng, không quản lý máy chủ |
| SaaS | xa-ét | Software as a Service | Dùng phần mềm chạy sẵn qua trình duyệt |
| Serverless | xơ-vơ-lét | Không máy chủ | Chạy hàm theo sự kiện, không quản lý máy chủ trực tiếp |
| Container | con-ten-nơ | Container | Gói ứng dụng cùng phụ thuộc, chạy cô lập trên hệ điều hành chung |
| Docker | đốc-cơ | Nền tảng container | Công cụ tạo và chạy container phổ biến |
| Image | i-mịch | Image | Bản mẫu chỉ đọc để tạo container |
| Dockerfile | đốc-cơ-phai | Tệp định nghĩa image | Công thức từng bước để build image |
| Registry | rét-gi-x-tri | Kho image | Nơi lưu và chia sẻ image, ví dụ Docker Hub |
| Docker Compose | đốc-cơ com-pốt | Công cụ chạy nhiều container | Mô tả và chạy một nhóm dịch vụ bằng tệp YAML |
| Kubernetes (K8s) | cu-bơ-nê-tết | Nền tảng điều phối container | Tự động triển khai, mở rộng và quản lý container |
| Pod | pót | Pod | Đơn vị nhỏ nhất trong Kubernetes, chứa một hoặc nhiều container |
| Node | nốt | Nút | Máy (vật lý hoặc ảo) chạy workload trong cluster |
| Cluster | clát-tơ | Cụm | Nhóm máy hoạt động cùng nhau |
| Deployment | đi-plôi-mần | Triển khai | Tài nguyên Kubernetes quản lý số bản sao của pod |
| Service | xơ-vít | Dịch vụ | Tài nguyên cung cấp địa chỉ ổn định cho nhóm pod |
| Ingress | ín-grết | Cổng vào | Định tuyến lưu lượng HTTP từ bên ngoài vào các service |
| Helm | hem | Trình quản lý gói Kubernetes | Đóng gói và triển khai ứng dụng Kubernetes bằng chart |
| Namespace | nêm-x-pêch | Không gian tên | Phân vùng logic trong một cluster |
| ConfigMap | con-phíc map | Bản đồ cấu hình | Lưu cấu hình không bí mật cho ứng dụng |
| Secret | xi-crít | Bí mật | Lưu thông tin nhạy cảm như mật khẩu, token |
| Monitoring | mon-ít-ring | Giám sát | Theo dõi trạng thái và hiệu năng hệ thống |
| Observability | ớp-xơ-vơ-bi-li-ti | Khả năng quan sát | Mức độ hiểu được trạng thái bên trong qua log, metric, trace |
| Log | lóc | Nhật ký | Bản ghi sự kiện theo thời gian |
| Metric | mé-trích | Chỉ số đo | Giá trị số đo theo thời gian như CPU, độ trễ |
| Trace | trết | Vết theo dõi | Đường đi của một yêu cầu qua nhiều dịch vụ |
| OpenTelemetry | âu-pần tơ-le-mê-tri | Chuẩn quan sát mở | Bộ API và công cụ thu thập log, metric, trace theo chuẩn mở |
| Uptime | ắp-taim | Thời gian hoạt động | Tỷ lệ thời gian dịch vụ hoạt động bình thường |
| SLA / SLO / SLI | ét-eo-ây / ét-eo-ô / ét-eo-ai | Thỏa thuận, mục tiêu, chỉ số mức dịch vụ | SLA là cam kết, SLO là mục tiêu nội bộ, SLI là số đo thực tế |
| Incident | in-xi-đần | Sự cố | Sự kiện làm gián đoạn hoặc giảm chất lượng dịch vụ |
| Rollback | rôn-bếch | Khôi phục phiên bản cũ | Quay lại phiên bản trước khi có sự cố |
| Canary release | ca-nây ri-lít | Phát hành canary | Triển khai cho một phần nhỏ người dùng trước khi mở rộng |
| Blue-green deployment | blu-grin đi-plôi-mần | Triển khai xanh-lục | Hai môi trường song song, chuyển lưu lượng khi sẵn sàng |
| Autoscaling | ô-tô-xcây-linh | Tự động mở rộng | Tự thêm hoặc bớt tài nguyên theo tải |
| Config drift | con-phíc đríp | Lệch cấu hình | Hạ tầng thực tế khác với cấu hình đã khai báo |

---

## Phần 6. Bảo mật thông tin

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| Authentication | ô-then-ti-cây-sần | Xác thực | Chứng minh bạn là ai, ví dụ bằng mật khẩu |
| Authorization | ô-tho-rai-dây-sần | Phân quyền | Quyết định người đã xác thực được làm gì |
| MFA | em-ép-ây | Xác thực đa yếu tố | Dùng từ hai yếu tố trở lên, ví dụ mật khẩu và mã OTP |
| OTP | âu-ti-pi | One-Time Password | Mật khẩu dùng một lần |
| SSO | ét-ét-âu | Single Sign-On | Đăng nhập một lần cho nhiều ứng dụng |
| OAuth | âu-âu-thờ | Giao thức ủy quyền | Cho phép ứng dụng truy cập tài nguyên thay người dùng mà không lộ mật khẩu |
| JWT | giây-đáp-liu-ti | JSON Web Token | Token mã hóa chứa thông tin xác thực, dùng trong API |
| Session | xéc-sần | Phiên làm việc | Trạng thái đăng nhập được duy trì giữa các yêu cầu |
| Cookie | cúc-ki | Cookie | Dữ liệu nhỏ trình duyệt lưu, thường dùng cho session |
| Encryption | en-crip-sần | Mã hóa | Biến dữ liệu thành dạng không đọc được nếu không có khóa |
| Hash | hết | Băm | Hàm biến dữ liệu thành chuỗi cố định, không đảo ngược được |
| Salt | xồn-t | Muối | Chuỗi ngẫu nhiên thêm vào trước khi băm mật khẩu |
| Symmetric encryption | xi-mê-tric en-crip-sần | Mã hóa đối xứng | Dùng cùng một khóa để mã hóa và giải mã |
| Asymmetric encryption | ây-xi-mê-tric en-crip-sần | Mã hóa bất đối xứng | Dùng cặp khóa công khai và khóa bí mật |
| Certificate | xơ-tí-phi-két | Chứng chỉ số | Tài liệu điện tử xác nhận danh tính khóa công khai |
| TLS | ti-eo-ét | Transport Layer Security | Giao thức mã hóa đường truyền, nền tảng của HTTPS |
| SSH | ét-ét-hát | Secure Shell | Giao thức đăng nhập và điều khiển máy từ xa an toàn |
| Zero trust | di-rô trớt | Không tin cậy mặc định | Kiểm tra mọi truy cập, không tin mạng nội bộ |
| Least privilege | lít-xt prí-vi-lịch | Đặc quyền tối thiểu | Chỉ cấp quyền đúng mức cần thiết |
| Vulnerability | van-nờ-ra-bi-li-ti | Lỗ hổng | Điểm yếu có thể bị khai thác |
| CVE | xi-vi-i | Common Vulnerabilities and Exposures | Mã định danh chuẩn cho lỗ hổng đã công bố |
| Patch | pét | Bản vá | Mã sửa lỗi hoặc lỗ hổng |
| Malware | mal-uea | Phần mềm độc hại | Phần mềm gây hại, gồm virus, ransomware, spyware |
| Ransomware | ran-xơm-uea | Mã đòi tiền chuộc | Mã hóa dữ liệu nạn nhân và đòi tiền để giải mã |
| Phishing | phi-sinh | Lừa đảo bằng mạo danh | Lừa người dùng cung cấp thông tin nhạy cảm |
| Social engineering | xô-xần en-gi-ni-rinh | Kỹ thuật xã hội | Thao túng con người thay vì tấn công kỹ thuật |
| SQL injection | ét-kiu-eo in-gi-sần | Tiêm mã SQL | Chèn mã SQL độc hại vào đầu vào của ứng dụng |
| XSS | ếch-ét-ét | Cross-Site Scripting | Chèn mã script độc hại vào trang web |
| DDoS | đi-đi-âu-ét | Distributed Denial of Service | Tấn công làm quá tải dịch vụ bằng nhiều nguồn |
| Penetration testing | pen-ơ-trây-sần tét-tinh | Kiểm thử xâm nhập | Thử tấn công có ủy quyền để tìm lỗ hổng |
| Audit log | ô-đít lóc | Nhật ký kiểm toán | Ghi lại ai đã làm gì và khi nào |
| Secrets management | xi-crít man-nờ-gi-mần | Quản lý bí mật | Lưu và cấp phát mật khẩu, khóa an toàn cho ứng dụng |
| Supply chain attack | xắp-plai chên ơ-tác | Tấn công chuỗi cung ứng | Tấn công qua thư viện hoặc công cụ phụ thuộc |

---

## Phần 7. Dữ liệu và thống kê

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| Data | đây-tơ | Dữ liệu | Thông tin thô, chưa được xử lý |
| Dataset | đây-tơ-xét | Tập dữ liệu | Tập hợp dữ liệu dùng cho phân tích hoặc huấn luyện |
| Feature | phi-trơ | Đặc trưng | Thuộc tính đầu vào của mô hình |
| Label | lê-bồ | Nhãn | Kết quả đúng mà mô hình học để dự đoán |
| Sample | xem-pờ | Mẫu | Một điểm dữ liệu trong tập |
| Training set | trây-ninh xét | Tập huấn luyện | Dữ liệu dùng để học |
| Validation set | va-li-đây-sần xét | Tập kiểm định | Dữ liệu dùng để chọn siêu tham số |
| Test set | test xét | Tập kiểm thử | Dữ liệu dùng để đánh giá cuối cùng |
| Data cleaning | đây-tơ clin-inh | Làm sạch dữ liệu | Sửa, loại bỏ dữ liệu sai, thiếu hoặc trùng |
| Outlier | au-tlai-ơ | Giá trị ngoại lai | Giá trị lệch bất thường so với phần còn lại |
| Mean | min | Trung bình | Tổng chia số lượng |
| Median | mi-đi-ân | Trung vị | Giá trị ở giữa khi sắp xếp |
| Standard deviation | xtén-đờ-đi-ét-i-ây-sần | Độ lệch chuẩn | Đo mức độ phân tán của dữ liệu quanh trung bình |
| Variance | ve-ri-ần | Phương sai | Bình phương độ lệch chuẩn |
| Correlation | cô-ri-lây-sần | Tương quan | Mức độ hai biến cùng biến đổi với nhau |
| Hypothesis testing | hai-pô-tê-xít tét-tinh | Kiểm định giả thuyết | Quy trình kiểm tra giả thuyết bằng dữ liệu |
| p-value | pi-va-liu | Giá trị p | Xác suất thấy kết quả như vậy nếu giả thuyết không đúng |
| Overfitting | âu-vơ-phít-tinh | Quá khớp | Mô hình học quá sát dữ liệu huấn luyện và kém trên dữ liệu mới |
| Underfitting | ăn-đơ-phít-tinh | Thiếu khớp | Mô hình quá đơn giản, không học được quy luật |
| Dashboard | dét-bót | Bảng điều khiển | Giao diện tổng hợp chỉ số để theo dõi |
| Visualization | vi-giu-a-lai-dây-sần | Trực quan hóa | Biểu diễn dữ liệu bằng biểu đồ |

---

## Phần 8. Trí tuệ nhân tạo và học máy

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| AI | ây-ai | Trí tuệ nhân tạo | Ngành tạo hệ thống thực hiện công việc đòi hỏi trí thông minh |
| Machine Learning (ML) | mơ-sin lớt-ninh | Học máy | Nhánh AI học quy luật từ dữ liệu thay vì lập trình luật cứng |
| Deep Learning (DL) | đíp lớt-ninh | Học sâu | Học máy dùng mạng nơ-ron nhiều tầng |
| Neural network | niu-rồ nét-quớc | Mạng nơ-ron | Mô hình gồm các nút nối với nhau, học bằng cách điều chỉnh trọng số |
| Neuron | niu-rôn | Nơ-ron nhân tạo | Đơn vị tính toán nhận đầu vào, áp dụng trọng số và hàm kích hoạt |
| Weight | uét | Trọng số | Tham số mô hình học được, quyết định ảnh hưởng của đầu vào |
| Bias | bai-ơ-xơ | Độ chệch | Tham số thêm vào để dịch chuyển đầu ra. Cũng là thuật ngữ chỉ thiên lệch dữ liệu |
| Activation function | ắc-tí-vây-sần phăng-sần | Hàm kích hoạt | Hàm phi tuyến quyết định đầu ra của nơ-ron, ví dụ ReLU |
| Loss function | lót phăng-sần | Hàm mất mát | Đo độ sai lệch giữa dự đoán và đáp án đúng |
| Gradient descent | grết-đi-ần đi-xen | Gradient descent | Thuật toán điều chỉnh tham số theo hướng giảm mất mát |
| Backpropagation | béc-prô-pơ-ghây-sần | Lan truyền ngược | Tính gradient của mất mát theo từng tham số để cập nhật |
| Epoch | ép-pốc | Epoch | Một lần duyệt qua toàn bộ tập huấn luyện |
| Batch | bét | Lô dữ liệu | Nhóm mẫu xử lý cùng lúc khi huấn luyện |
| Learning rate | lớt-ninh rết | Tốc độ học | Độ lớn bước điều chỉnh tham số mỗi lần cập nhật |
| Hyperparameter | hai-pơ-pa-ram-i-tơ | Siêu tham số | Tham số chọn trước khi huấn luyện, ví dụ learning rate, số tầng |
| Supervised learning | xu-pơ-vai-dơ lớt-ninh | Học có giám sát | Học từ dữ liệu có nhãn |
| Unsupervised learning | ăn-xu-pơ-vai-dơ lớt-ninh | Học không giám sát | Tìm cấu trúc trong dữ liệu không có nhãn |
| Reinforcement learning | ri-in-phớt-x-mần lớt-ninh | Học tăng cường | Học qua thử và sai, nhận phần thưởng từ môi trường |
| Classification | clát-xi-phi-cây-sần | Phân loại | Dự đoán nhãn rời rạc, ví dụ spam hay không |
| Regression | ri-grét-sần | Hồi quy | Dự đoán giá trị số liên tục, ví dụ giá nhà |
| Clustering | clát-tơ-ring | Phân cụm | Gom các mẫu tương tự thành nhóm |
| Decision tree | đi-xi-dần trí | Cây quyết định | Mô hình ra quyết định theo chuỗi điều kiện dạng cây |
| Random forest | ren-đầm pho-rét | Rừng ngẫu nhiên | Tập hợp nhiều cây quyết định |
| SVM | ét-vi-em | Support Vector Machine | Mô hình tìm siêu phẳng phân tách các lớp |
| CNN | xi-en-en | Convolutional Neural Network | Mạng nơ-ron tích chập, mạnh cho ảnh |
| RNN | a-en-en | Recurrent Neural Network | Mạng nơ-ron hồi quy, xử lý chuỗi có thứ tự |
| LSTM | eo-ét-ét-em | Long Short-Term Memory | Biến thể RNN giảm vấn đề quên thông tin xa |
| Transformer | tran-xphoóc-mơ | Transformer | Kiến trúc dùng cơ chế attention, nền tảng của LLM hiện nay |
| Attention | ơ-ten-sần | Cơ chế chú ý | Cho phép mô hình tập trung vào các phần quan trọng của đầu vào |
| Embedding | em-bét-đinh | Nhúng | Biểu diễn đối tượng bằng vector số trong không gian nhiều chiều |
| Vector | vét-tơ | Vector | Mảng số biểu diễn dữ liệu hoặc ý nghĩa |
| Tokenization | tô-kần-i-dây-sần | Tách token | Chia văn bản thành các đơn vị nhỏ (token) để mô hình xử lý |
| Token | tô-kần | Token | Đơn vị văn bản mà mô hình đọc và sinh ra, có thể là từ hoặc một phần từ |
| Computer vision | com-piu-tơ vi-dần | Thị giác máy tính | Nhánh AI xử lý và hiểu ảnh, video |
| NLP | en-eo-pi | Natural Language Processing | Xử lý ngôn ngữ tự nhiên |
| Speech recognition | xpít rê-cơ-nì-sần | Nhận dạng giọng nói | Chuyển âm thanh thành văn bản |
| Object detection | óp-dếch đi-téc-sần | Phát hiện đối tượng | Xác định vị trí và loại đối tượng trong ảnh |
| Transfer learning | trens-phơ lớt-ninh | Học chuyển giao | Dùng mô hình đã huấn luyện làm điểm khởi đầu cho bài toán khác |
| Fine-tuning | phai-tiu-ninh | Tinh chỉnh | Huấn luyện thêm mô hình đã có trên dữ liệu chuyên biệt |
| Pre-training | pri-trây-ninh | Tiền huấn luyện | Huấn luyện ban đầu trên dữ liệu lớn và tổng quát |
| Inference | in-phơ-răn-x | Suy luận | Chạy mô hình đã huấn luyện để tạo đầu ra |
| Model drift | mô-đồ đríp | Trôi dạt mô hình | Hiệu năng giảm khi dữ liệu thực tế thay đổi so với lúc huấn luyện |
| Ensemble | an-xăm-bồ | Tập hợp mô hình | Kết hợp nhiều mô hình để có kết quả tốt hơn |
| Confusion matrix | con-phiu-dần mét-rích | Ma trận nhầm lẫn | Bảng đếm đúng sai theo từng lớp |
| Precision | pri-xi-dần | Độ chính xác dương | Trong các dự đoán dương, bao nhiêu là đúng |
| Recall | ri-cô | Độ phủ | Trong các mẫu dương thật, mô hình tìm được bao nhiêu |
| F1-score | ép-oan xcô | Điểm F1 | Trung bình điều hòa của precision và recall |
| Accuracy | ắc-kiu-ră-si | Độ chính xác | Tỷ lệ dự đoán đúng trên tổng số |
| AUC-ROC | ây-iu-xi | Diện tích dưới đường ROC | Đo khả năng phân biệt các lớp của mô hình |
| Bias-variance tradeoff | bai-ơ-xơ va-ri-ần trây-đóp | Đánh đổi độ chệch và phương sai | Cân bằng giữa mô hình quá đơn giản và quá phức tạp |
| Cross-validation | crốt va-li-đây-sần | Kiểm định chéo | Chia dữ liệu nhiều lần để đánh giá ổn định hơn |
| Feature engineering | phi-trơ en-gi-ni-rinh | Kỹ thuật đặc trưng | Tạo và chọn đặc trưng để mô hình học tốt hơn |
| GPU training | gi-pi-iu trây-ninh | Huấn luyện trên GPU | Dùng GPU để tăng tốc huấn luyện |
| Explainable AI (XAI) | ép-xơ-plê-na-bồ ây-ai | AI có thể giải thích | Phương pháp giúp con người hiểu lý do mô hình đưa ra quyết định |
| Bias (đạo đức AI) | bai-ơ-xơ | Thiên lệch | Mô hình đưa kết quả bất công do dữ liệu hoặc thiết kế |
| Hallucination | ha-lu-xi-nây-sần | Ảo giác | Mô hình tạo thông tin nghe hợp lý nhưng sai sự thật |

---

## Phần 9. Mô hình ngôn ngữ lớn và AI tạo sinh

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| LLM | eo-eo-em | Large Language Model | Mô hình ngôn ngữ lớn, huấn luyện trên lượng văn bản khổng lồ |
| GenAI | gen-ây-ai | Generative AI | AI tạo nội dung mới như văn bản, ảnh, âm thanh |
| Foundation model | phao-đây-sần mô-đồ | Mô hình nền tảng | Mô hình lớn, tổng quát, dùng làm cơ sở cho nhiều ứng dụng |
| Parameter | pa-ram-i-tơ | Tham số | Các trọng số học được. Số lượng tham số thường gắn với quy mô mô hình |
| Context window | con-tếch-x uin-đâu | Cửa sổ ngữ cảnh | Lượng token tối đa mô hình xử lý trong một lần |
| Prompt | prăm-pt | Lời nhắc | Đầu vào hướng dẫn mô hình làm việc |
| System prompt | xi-xtem prăm-pt | Lời nhắc hệ thống | Chỉ dẫn cố định định hình hành vi mô hình |
| Prompt engineering | prăm-pt en-gi-ni-rinh | Kỹ thuật viết lời nhắc | Thiết kế lời nhắc để có kết quả tốt hơn |
| Few-shot | phiu-sốt | Vài ví dụ | Đưa vài ví dụ mẫu vào lời nhắc |
| Zero-shot | di-rô-sốt | Không ví dụ | Yêu cầu mô hình làm việc mà không có ví dụ |
| Chain-of-thought | trên-ốp-thót | Chuỗi suy nghĩ | Yêu cầu mô hình trình bày từng bước suy luận |
| Temperature | tem-pơ-rơ-trơ | Nhiệt độ | Tham số điều chỉnh mức ngẫu nhiên của đầu ra. Cao hơn thì đa dạng hơn |
| Top-p | tóp pi | Nucleus sampling | Chỉ chọn trong nhóm token có tổng xác suất đạt ngưỡng p |
| Sampling | xem-pling | Lấy mẫu | Chọn token tiếp theo theo phân bố xác suất |
| Greedy decoding | grít-đi đi-cô-đinh | Giải mã tham lam | Luôn chọn token có xác suất cao nhất |
| Hallucination | ha-lu-xi-nây-sần | Ảo giác | Đã nêu ở Phần 8. Thường gặp khi mô hình thiếu căn cứ |
| Grounding | grao-đinh | Neo vào nguồn | Gắn câu trả lời với dữ liệu hoặc nguồn có thể kiểm chứng |
| Alignment | ơ-lai-n-mần | Căn chỉnh | Làm cho mô hình hành xử đúng ý người dùng và giá trị mong muốn |
| RLHF | a-eo-ét-ếch-ép | Học tăng cường từ phản hồi con người | Huấn luyện mô hình dựa trên đánh giá của người |
| Instruction tuning | in-xtrắc-sần tiu-ninh | Tinh chỉnh theo chỉ dẫn | Huấn luyện mô hình để làm theo chỉ dẫn tự nhiên |
| Multimodal | mon-ti-mô-đồ | Đa phương thức | Mô hình xử lý nhiều loại dữ liệu như văn bản, ảnh, âm thanh |
| Quantization | quan-tí-dây-sần | Lượng tử hóa | Giảm độ chính xác số để mô hình nhẹ và chạy nhanh hơn |
| Distillation | đít-tin-lây-sần | Chưng cất mô hình | Huấn luyện mô hình nhỏ bắt chước mô hình lớn |
| Open-weight model | âu-pần uét mô-đồ | Mô hình trọng số mở | Mô hình có công bố trọng số để tải và chạy, khác với mã nguồn mở đầy đủ |
| Token cost | tô-kần cót | Chi phí token | Chi phí tính theo số token đầu vào và đầu ra |
| Latency (LLM) | lây-ten-xi | Độ trễ phản hồi | Thời gian từ khi gửi yêu cầu đến khi nhận phản hồi |
| Streaming | xtrim-ming | Truyền luồng | Nhận phản hồi từng phần khi mô hình đang sinh |
| Knowledge cutoff | nơ-lét cắt-phơ | Mốc kiến thức | Thời điểm cuối của dữ liệu huấn luyện, mô hình không biết thông tin sau đó |
| Diffusion model | đi-phiu-dần mô-đồ | Mô hình khuếch tán | Mô hình tạo ảnh bằng cách khử nhiễu dần |
| Text-to-image | tếch-tu-i-mết | Văn bản sang ảnh | Tạo ảnh từ mô tả bằng văn bản |

---

## Phần 10. Agent, RAG và giao thức tích hợp

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| AI Agent | ây-ai ê-jần | Tác nhân AI | Hệ thống dùng LLM để lập kế hoạch, gọi công cụ và thực hiện nhiều bước |
| Agentic | ê-jần-tích | Mang tính tác nhân | Đặc tính của hệ thống hành động tự chủ theo mục tiêu |
| Tool use | tu-lơ diu | Sử dụng công cụ | Mô hình gọi hàm hoặc API bên ngoài để làm việc |
| Function calling | phăng-sần côl-linh | Gọi hàm | Mô hình sinh tham số có cấu trúc để ứng dụng gọi hàm tương ứng |
| Orchestration | o-rét-xtrây-sần | Điều phối | Quản lý luồng công việc giữa nhiều bước, công cụ, agent |
| Planning | plen-ninh | Lập kế hoạch | Chia mục tiêu thành các bước thực hiện |
| Memory (agent) | mem-ơ-ri | Bộ nhớ agent | Lưu trạng thái và kinh nghiệm để dùng ở các bước sau |
| Human-in-the-loop | hiu-men in đơ lúp | Con người trong vòng lặp | Yêu cầu người duyệt ở bước quan trọng |
| Multi-agent | mơn-ti ê-jần | Đa tác nhân | Nhiều agent phối hợp để hoàn thành nhiệm vụ |
| RAG | rét | Retrieval-Augmented Generation | Lấy tài liệu liên quan rồi đưa vào ngữ cảnh để LLM trả lời có căn cứ |
| Retrieval | ri-tri-vồ | Truy hồi | Tìm các đoạn tài liệu liên quan đến câu hỏi |
| Chunking | chân-kinh | Chia đoạn | Cắt tài liệu thành các phần nhỏ để lập chỉ mục |
| Semantic search | xơ-man-tích xéc-trờ | Tìm kiếm ngữ nghĩa | Tìm theo ý nghĩa nhờ vector, không chỉ khớp từ khóa |
| Reranking | ri-ren-kinh | Xếp hạng lại | Sắp xếp lại kết quả tìm được theo mức liên quan |
| Knowledge base | nô-lét bây-x | Cơ sở tri thức | Kho tài liệu và thông tin mà hệ thống dùng để trả lời |
| MCP | em-xi-pi | Model Context Protocol | Giao thức chuẩn để ứng dụng AI kết nối với công cụ và dữ liệu bên ngoài |
| MCP server | em-xi-pi xơ-vơ | Máy chủ MCP | Chương trình cung cấp công cụ, tài nguyên, lời nhắc qua MCP |
| MCP client | em-xi-pi cờ-lai-ân | Máy khách MCP | Thành phần trong ứng dụng AI kết nối đến MCP server |
| Tool | tu-lơ | Công cụ | Chức năng mà agent hoặc mô hình có thể gọi |
| JSON | giây-xơn | JavaScript Object Notation | Định dạng dữ liệu văn bản phổ biến để trao đổi |
| JSON Schema | giây-xơn xki-ma | Lược đồ JSON | Mô tả cấu trúc và kiểu của dữ liệu JSON |
| Guardrails | gác-đrêlz | Rào chắn | Quy tắc hoặc bộ lọc giới hạn hành vi và đầu ra của AI |
| Prompt injection | prăm-pt in-giếch-sần | Tiêm lệnh | Chèn chỉ dẫn độc hại vào dữ liệu để đổi hành vi mô hình |
| Jailbreak | gio-breo-cờ | Bẻ khóa mô hình | Cố vượt qua giới hạn an toàn của mô hình |
| Sandbox | xen-bóc-x | Hộp cát | Môi trường cô lập để chạy mã hoặc công cụ không tin cậy |
| Agent loop | ê-jần lúp | Vòng lặp agent | Chu trình: suy nghĩ, gọi công cụ, quan sát kết quả, lặp lại |
| Hallucinated tool call | ha-lu-xi-nây-tíd tu-lơ côl | Gọi công cụ bịa | Mô hình gọi công cụ không tồn tại hoặc sai tham số |

---

## Phần 11. Vận hành và đánh giá AI

| Thuật ngữ | Phiên âm | Nghĩa | Giải thích |
|---|---|---|---|
| MLOps | em-eo-ốp | Vận hành học máy | Thực hành đưa mô hình ML vào sản xuất và vận hành ổn định |
| LLMOps | eo-eo-em-ốp | Vận hành LLM | Quản lý prompt, phiên bản mô hình, chi phí, đánh giá cho LLM |
| Model registry | mô-đồ rét-gi-x-tri | Kho mô hình | Nơi lưu và quản lý phiên bản mô hình |
| Feature store | phi-trơ xtô | Kho đặc trưng | Nơi lưu và cấp đặc trưng đồng nhất cho huấn luyện và suy luận |
| Experiment tracking | ếch-xpe-ri-mần trếch-king | Theo dõi thí nghiệm | Ghi lại tham số, kết quả, mã để so sánh các lần huấn luyện |
| Evaluation (eval) | i-van-lu-ây-sần | Đánh giá | Đo chất lượng mô hình hoặc hệ thống theo tiêu chí |
| Benchmark | ben-sờ-mác | Bộ chuẩn đo | Tập bài kiểm tra chuẩn để so sánh mô hình |
| Golden dataset | gôn-đần đây-tơ-xét | Bộ dữ liệu chuẩn vàng | Tập dữ liệu có đáp án đúng đã được kiểm duyệt để đánh giá |
| A/B testing | ây-bi tét-tinh | Thử nghiệm A/B | So sánh hai phiên bản bằng cách chia người dùng ngẫu nhiên |
| Drift detection | đríp đi-tếch-sần | Phát hiện trôi dạt | Phát hiện dữ liệu hoặc hiệu năng thay đổi theo thời gian |
| Hallucination rate | ha-lu-xi-nây-sần rết | Tỷ lệ ảo giác | Tỷ lệ câu trả lời sai sự thật trên tập đánh giá |
| Latency budget | lây-ten-xi bát-jết | Ngân sách độ trễ | Giới hạn thời gian chấp nhận được cho một yêu cầu |
| Cost per request | cót pơ ri-kuét | Chi phí mỗi yêu cầu | Chi phí trung bình của một lần gọi |
| Rate limit | rết lim-it | Giới hạn tốc độ | Số yêu cầu tối đa trong một khoảng thời gian |
| Quota | quâu-tơ | Hạn mức | Tổng lượng tài nguyên được phép dùng |
| Caching (LLM) | kế-sinh | Lưu đệm | Lưu kết quả để trả lời nhanh các yêu cầu lặp lại |
| Observability (AI) | ớp-xơ-vơ-bi-li-ti | Khả năng quan sát AI | Theo dõi prompt, phản hồi, token, lỗi và độ trễ |
| Red teaming | rét ti-minh | Thử tấn công đội đỏ | Thử có chủ đích các cách làm AI sinh nội dung không an toàn |
| Safety filter | xây-phờ-ti phin-tơ | Bộ lọc an toàn | Lọc nội dung đầu vào và đầu ra có hại |
| Data governance | đây-tơ gốp-vơ-nần | Quản trị dữ liệu | Quy định ai được truy cập, dùng và chịu trách nhiệm về dữ liệu |
| Model card | mô-đồ các | Thẻ mô hình | Tài liệu mô tả mục đích, dữ liệu, giới hạn và đánh giá của mô hình |

---

## Ghi chú về phiên âm và nghĩa

- Phiên âm trong tài liệu là cách đọc gần đúng cho người đọc tiếng Việt. Một số từ (đặc biệt là thuật ngữ mới) có nhiều cách đọc trong thực tế. Phiên âm IPA chuẩn không được đưa vào để tránh sai sót.
- Một số thuật ngữ xuất hiện ở nhiều phần với ngữ cảnh khác nhau, ví dụ "Bias" trong học máy và "Bias" về đạo đức AI, hoặc "Latency" trong mạng và trong LLM. Tài liệu ghi chú ngữ cảnh khi cần.
- Các định nghĩa là tóm tắt ngắn. Muốn hiểu sâu, nên đọc các tài liệu chi tiết hơn của từng chủ đề.
- Chưa đối chiếu từng định nghĩa với tài liệu chính thức trong lần viết này. Nếu cần chính xác cho mục đích học thuật, nên kiểm tra lại với nguồn gốc.
