---
trigger: always_on
description: "Quy tắc tự động kích hoạt chuẩn vẽ biểu đồ phần mềm (Use Case, Sequence, ERD, Class, Flowchart) đồng bộ mã nguồn thật và xuất ra Desktop/docs khi người dùng yêu cầu biểu đồ."
---

# Software Diagram Architect Rule

## Quy Tắc Bắt Buộc Khi Thiết Kế & Vẽ Biểu Đồ Phần Mềm

Bất kỳ khi nào người dùng yêu cầu vẽ hoặc cập nhật biểu đồ:
- *"vẽ use case"*, *"vẽ sơ đồ ca sử dụng"*, *"vẽ ERD"*, *"vẽ sơ đồ cơ sở dữ liệu"*
- *"vẽ sequence"*, *"biểu đồ tuần tự"*, *"vẽ class diagram"*, *"vẽ kiến trúc hệ thống"*
- Hoặc yêu cầu trực quan hóa luồng nghiệp vụ của bất kỳ chức năng nào trong codebase:

Agent **BẮT BUỘC** phải tuân thủ 100% quy trình 2 bước dưới đây:

---

### BƯỚC 1: XÁC ĐỊNH PHẠM VI & ĐỊNH DẠNG ĐẦU RA QUA `grill-me`
Trước khi vẽ, Agent gọi công cụ `ask_question` để người dùng xác nhận 2 lựa chọn:
1. **Phạm vi vẽ (Scope)**:
   - Toàn bộ hệ thống (System-wide overview).
   - Hoặc chỉ 1 phân hệ / chức năng cụ thể (ví dụ: Đăng nhập & Xác thực, Quét mã QR gọi món, Thanh toán hóa đơn, Quản lý kho nguyên liệu...).
2. **Định dạng kết xuất (Output Format)**:
   - `(Recommended) Đa định dạng (Mermaid hiển thị trên Chat/Markdown + PlantUML/DBML code + file .drawio)`
   - `Chỉ hiển thị trực tiếp biểu đồ Mermaid trên Markdown`

---

### BƯỚC 2: ĐỐI SOÁT SOURCE CODE THẬT & KẾT XUẤT 2 NƠI (PROJECT + DESKTOP)
Sau khi người dùng chọn trên Interactive Question Box:

1. **Khảo sát Codebase (Code-grounded)**:
   - Không bịa đặt: Đọc trực tiếp Entity/Model, Servlet, Filter, Service, DAO và SQL script tương ứng trong codebase `capheQR-main`.
   - Lấy đúng tên bảng, tên trường khóa chính (`PK`), khóa ngoại (`FK`), tên biến, tên hàm, đường dẫn URL API thực tế.

2. **Áp dụng Chuẩn Chống Lỗi Hiển Thị**:
   - Mọi nhãn tiếng Việt hoặc chuỗi có ký tự đặc biệt (`/`, `(`, `)`, `:`, `[`, `]`) bắt buộc bọc trong dấu ngoặc kép `"..."`.
   - Node ID viết liền không dấu bằng tiếng Anh chuẩn.

3. **Lưu Trữ Đồng Bộ Cả 2 Vị Trí**:
   - Vị trí 1 (Dự án): Lưu file vào `docs/diagrams/<loai-bieu-do>.md`.
   - Vị trí 2 (Desktop): Tự động tạo và cập nhật bản sao tại thư mục Desktop của người dùng:
     👉 `C:\Users\ad\Desktop\Diagrams_CapheQR/<loai-bieu-do>.*`
     *(bao gồm file `.md`, file `.drawio` hoặc file code PlantUML `.puml` để kéo thả trực tiếp vào Microsoft Word hoặc diagrams.net)*.
   - Hiển thị trực tiếp khối biểu đồ hoàn chỉnh trên khung chat để người dùng xem trước tức thì.

