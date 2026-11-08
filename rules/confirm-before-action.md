---
trigger: always_on
description: "Bắt buộc tự động kích hoạt skill grill-me và hiển thị Interactive Question Box (ask_question) để chốt phương án và phê duyệt trước khi viết code."
---

# Mandatory Pre-Execution Approval via `grill-me` Interactive Box (Phase 0: Approval Gate)

Khi nhận bất kỳ yêu cầu mới nào từ người dùng (tạo tính năng, sửa bug, refactor, thiết kế UI/API, hoặc cài đặt thư viện):

## BƯỚC 1: PHÂN TÍCH NGỮ CẢNH (READ-ONLY)
- Đọc file ngữ cảnh (`.antigravity/activeContext.md`) và các file liên quan trong codebase.
- **TUYỆT ĐỐI KHÔNG** tự ý viết/sửa code, tạo file mới, xóa file hoặc chạy lệnh terminal làm thay đổi hệ thống ngay lập tức.

## BƯỚC 2: KÍCH HOẠT TỰ ĐỘNG SKILL `grill-me` (INTERACTIVE BOX)
- **BẮT BUỘC** gọi công cụ `ask_question` theo quy chuẩn của skill `grill-me`. Tuyệt đối không chỉ hỏi bằng câu hỏi văn bản thuần túy ở cuối câu trả lời.
- **Trình bày tóm tắt phương án trước hộp câu hỏi**:
  1. *Giải pháp đề xuất*: Kiến trúc, luồng xử lý và các tiêu chuẩn bảo mật (`hardened-backend-security`) hoặc UI/UX (`ui-ux-pro-architect`) tương ứng.
  2. *Danh sách file dự kiến tác động*: Ghi rõ file nào sẽ tạo mới (`[+]`), chỉnh sửa (`[*]`), hoặc xóa (`[-]`).
- **Hiển thị Hộp câu hỏi tương tác (Interactive Question Box)**:
  * Gọi `ask_question` với 1 đến 3 câu hỏi trắc nghiệm ngắn gọn để làm rõ:
    - Quyết định kiến trúc / Tech stack / Thư viện.
    - Xử lý Edge cases, Security hoặc UX trade-offs.
    - Câu hỏi phê duyệt kế hoạch (Plan Approval Gate): Xin ý kiến chốt để bắt đầu viết code.
  * Mỗi câu hỏi cần có các lựa chọn gợi ý sẵn (Options), phương án tối ưu nhất đặt lên đầu tiên với tiền tố `(Recommended)`. Giao diện UI sẽ tự động cung cấp tùy chọn "Other / Nhập câu trả lời khác" cho người dùng.

## BƯỚC 3: CHỜ NGƯỜI DÙNG CHỌN TRÊN BOX (HARD STOP)
- Dừng lại hoàn toàn (kết thúc lượt tương tác của tool) cho đến khi người dùng submit câu trả lời từ Interactive Question Box.
- **Sau khi người dùng phê duyệt phương án**: Cổng phê duyệt mở khóa (Unblocked). Chuyển tiếp ngay sang quy trình Spec Kit (`speckit-specify` -> `speckit-plan` -> `speckit-tasks` -> `speckit-implement`) để tạo đặc tả và triển khai code theo đúng phương án đã chốt.

---

## NGOẠI LỆ HỢP LỆ (EXCEPTIONS)
1. Cập nhật 2 file bộ nhớ (`.antigravity/activeContext.md` và `.antigravity/progress.md`) sau khi đã hoàn thành task được duyệt.
2. Quy trình Spec Kit sau khi đã qua Cổng phê duyệt (được phép tạo file đặc tả trong `.specify/`).
