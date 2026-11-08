---
name: software-diagram-architect
description: "Chuẩn hóa khả năng phân tích source code thực tế để tự động thiết kế và vẽ các biểu đồ phần mềm (Use Case, Sequence, ERD, Class, Architecture) chuẩn học thuật, kỹ thuật và xuất ra Desktop/docs."
---

# Software Diagram Architect Skill

Kỹ năng chuyên gia kiến trúc và vẽ biểu đồ phần mềm (Use Case, Sequence, ERD, Class, System Architecture, Flowchart) đạt chuẩn học thuật (đồ án, báo cáo công nghệ) và chuẩn kỹ thuật doanh nghiệp, đảm bảo đồng bộ 100% với mã nguồn thực tế và kết xuất không bao giờ bị lỗi hiển thị.

---

## 1. QUY TẮC ĐIỀU HƯỚNG CÔNG CỤ THEO LOẠI BIỂU ĐỒ (TOOL ROUTING)

### A. Use Case Diagram (Biểu đồ Ca sử dụng)
- **Ưu tiên hàng đầu**: Xuất bằng **PlantUML** (`@startuml ... @enduml`) hoặc tạo file **`.drawio`** XML chuẩn.
  * Thể hiện rõ Actor người que (`:Khách hàng:`, `:Nhân viên:`, `:Admin:`).
  * System Boundary trực quan dạng `rectangle "Hệ Thống Quản Lý Cà Phê QR" { ... }`.
  * Đúng chuẩn quan hệ UML: Kế thừa `--|>`, bao gồm `.> (UC_X) : <<include>>`, mở rộng `.> (UC_Y) : <<extend>>`.
- **Nếu hiển thị trực tiếp trên chat/Markdown**: Sử dụng `flowchart LR` của Mermaid kết hợp `subgraph` làm ranh giới hệ thống, chú thích đường đứt nét: `UC_Order -.->|"<<include>>"| UC_CheckStock`.

### B. Entity Relationship Diagram (ERD - Biểu đồ Thực thể Liên kết)
- **Công cụ**: Sử dụng `erDiagram` của **Mermaid** hoặc mã nguồn **DBML** (để import 1-click vào [dbdiagram.io](https://dbdiagram.io)).
- **Tiêu chuẩn**:
  * Phải ghi rõ Khóa chính (`PK`), Khóa ngoại (`FK`).
  * Ghi rõ kiểu dữ liệu thực tế (`int`, `nvarchar(100)`, `decimal(10,2)`, `datetime`, `boolean`).
  * Thể hiện chính xác bản số quan hệ (Cardinality): `||--o{` (1-nhiều), `||--||` (1-1), `}|--|{` (nhiều-nhiều).
  * Thêm chú thích mô tả ý nghĩa trường tiếng Việt bằng dấu ngoặc kép.

### C. Sequence Diagram (Biểu đồ Tuần tự Luồng Nghiệp vụ)
- **Công cụ**: Sử dụng `sequenceDiagram` của **Mermaid**.
- **Tiêu chuẩn bắt buộc**:
  * Luôn bật `autonumber` để đánh số thứ tự các bước giao tiếp.
  * Phân tách rõ ràng ít nhất 4 đến 5 tầng đối tượng (Participants/Actors):
    `Actor (Người dùng)` $\rightarrow$ `Frontend/UI (JSP/Web)` $\rightarrow$ `API/Controller (Servlet)` $\rightarrow$ `Service / DAO` $\rightarrow$ `Database (SQL Server)`.
  * **Xử lý rẽ nhánh đầy đủ (Flow Integrity)**: Bắt buộc dùng khối `alt ... else ... end` để mô tả cả luồng thành công (Happy Path) và luồng bắt lỗi/thất bại (Error Handling/Exception).
  * Sử dụng kích hoạt thực thi: `activate` và `deactivate` trên các đối tượng.

### D. Class Diagram / System Architecture / Flowchart
- **Công cụ**: Sử dụng `classDiagram` hoặc `flowchart TD / LR` của **Mermaid**.
- **Tiêu chuẩn Class**: Thể hiện rõ phạm vi truy cập (`+` public, `-` private, `#` protected), các thuộc tính và phương thức chính, quan hệ kế thừa (`<|--`), hiện thực hóa (`<|..`), kết tập (`o--`), hợp thành (`*--`).

---

## 2. NGUYÊN TẮC CỐT LÕI: ĐỒNG BỘ 100% VỚI SOURCE CODE THẬT (CODE-GROUNDED)

- **TUYỆT ĐỐI KHÔNG vẽ biểu đồ giả lập/bịa đặt**: Mọi thực thể, ca sử dụng hay hàm gọi trong biểu đồ phải được đọc và đối soát trực tiếp từ codebase:
  * **ERD & Class**: Đọc trực tiếp các file Entity/Model trong `src/main/java/.../model/`, cấu trúc CSDL trong file script SQL hoặc hibernate/JPA annotations.
  * **Sequence & Use Case**: Đọc trực tiếp các file Servlet/Controller trong `src/main/java/.../controller/`, các filter phân quyền, các hàm DAO và câu truy vấn thực tế.

---

## 3. CHỐNG LỖI HIỂN THỊ TUYỆT ĐỐI (ZERO RENDERING ERROR)

1. **Quy tắc bọc nhãn (Label Quoting)**:
   - Khi nhãn hiển thị có chứa Tiếng Việt có dấu hoặc bất kỳ ký tự đặc biệt nào (`/`, `\`, `(`, `)`, `[`, `]`, `{`, `}`, `:`, `-`, `&`, `|`): **BẮT BUỘC PHẢI BỌC TRONG DẤU NGOẶC KÉP `"..."`**.
   - *Ví dụ đúng*: `Actor_Guest["Khách Hàng (Vãng Lai)"] --> UC_Login(["Đăng Nhập (SĐT / Mật Khẩu)"])`.
2. **Quy tắc định danh Node ID**:
   - Node ID bắt buộc dùng tiếng Anh viết liền không dấu, có tiền tố phân loại: `UC_` (Use Case), `Tbl_` (Table), `Srv_` (Servlet), `DAO_` (Data Access Object).
3. **Lưu trữ tập trung và Xuất ra Desktop**:
   - Mọi biểu đồ sinh ra bắt buộc được lưu vào 2 vị trí:
     1. Thư mục dự án: `docs/diagrams/` (ví dụ: `docs/diagrams/usecase.md`, `docs/diagrams/erd.md`, `docs/diagrams/sequence.md`).
     2. Thư mục Desktop người dùng: `C:\Users\ad\Desktop\Diagrams_CapheQR\` để người dùng mở xem ngay, copy kéo thả vào Word/báo cáo đồ án.

