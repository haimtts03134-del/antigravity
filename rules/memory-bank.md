---
trigger: always_on
description: "Quy chuẩn bắt buộc về Persistent Memory Bank: Ngăn ngừa full codebase scan, tham chiếu context bắt buộc và tự động đồng bộ cuối phiên."
---

# Persistent Memory Bank Rules

Mục tiêu: Ngăn chặn việc quét toàn bộ repository (full-scan) mỗi khi bắt đầu task mới; tối ưu tốc độ phản hồi và tiết kiệm token context bằng cách quản lý bộ nhớ qua file cục bộ (`.antigravity/`).

---

## 1. Mandatory Context References

- **Bắt buộc đọc file ngữ cảnh**:
  - Khi bắt đầu bất kỳ task mới nào, nếu thư mục `.antigravity/` tồn tại trong workspace, Agent **BẮT BUỘC** phải đọc ngay hai file trước khi phân tích hay sinh code:
    1. `.antigravity/activeContext.md` (hiểu mục tiêu, trọng tâm hiện tại và các quyết định kỹ thuật).
    2. `.antigravity/progress.md` (nắm bắt tiến độ và trạng thái hiện tại của codebase).
  - Chỉ đọc thêm các file liên quan trực tiếp đến task đang được yêu cầu.
- **Nghiêm cấm Full Codebase Scan**:
  - Nghiêm cấm tuyệt đối việc chạy các lệnh tìm kiếm, duyệt file hoặc liệt kê diện rộng (như `find .`, `grep -r`, `Get-ChildItem -Recurse`, hoặc duyệt toàn bộ cây thư mục) khi người dùng không yêu cầu cụ thể.
  - Mọi thao tác truy xuất thông tin phải đi trúng đích (targeted reading) dựa trên các file đã được ghi nhận trong memory bank.

---

## 2. Session Wrap-up

- **Tự động đồng bộ trước khi kết thúc**:
  - Trước khi kết thúc lượt phản hồi của bất kỳ task nào, Agent **BẮT BUỘC** phải tự động cập nhật lại hai file:
    1. `.antigravity/activeContext.md`: Cập nhật `Current Focus` (trạng thái vừa thay đổi, điểm nhấn kỹ thuật) và `Next Steps` (các bước tiếp theo cần làm).
    2. `.antigravity/progress.md`: Đánh dấu hoàn thành `[x]` các task đã xong và cập nhật trạng thái hoạt động của codebase.
- **Tính liên tục của phiên (Session Continuity)**:
  - Đảm bảo lượt làm việc kế tiếp luôn có đầy đủ ngữ cảnh mới nhất từ memory bank mà không cần phải hỏi lại hoặc tái phân tích repository.
