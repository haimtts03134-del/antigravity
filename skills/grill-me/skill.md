---
name: grill-me
description: "Interactive requirement elicitation and plan approval interview: tự động gọi công cụ ask_question để hiển thị hộp câu hỏi trắc nghiệm tương tác (Interactive Question Box kiểu Claude) nhằm làm rõ quyết định kiến trúc, bảo mật, UX và phê duyệt kế hoạch trước khi viết code."
---

# Skill: Grill-Me Interactive Requirement & Plan Elicitation

Skill này hướng dẫn Agent cách thực hiện quy trình phỏng vấn làm rõ yêu cầu (Interview Elicitation) và phê duyệt kế hoạch thông qua hộp thoại tương tác đồ họa (**Interactive Question Box**) bằng công cụ `ask_question`.

---

## 1. Công Cụ Nền Tảng: `ask_question`

Antigravity cung cấp công cụ `ask_question` cho phép Agent dựng giao diện hộp câu hỏi trắc nghiệm tương tác (Multiple-choice UI) trực quan ngay trong khung chat:

### Schema Chuẩn Của `ask_question`:
```json
{
  "questions": [
    {
      "question": "Câu hỏi cần làm rõ?",
      "options": [
        "(Recommended) Phương án khuyến nghị tối ưu nhất kèm lý do ngắn gọn",
        "Phương án thay thế A",
        "Phương án thay thế B"
      ],
      "is_multi_select": false
    }
  ]
}
```

### Các Quy Tắc Hiển Thị Bắt Buộc Của UI:
1. **Không đánh số thứ tự thủ công**: UI của hệ thống sẽ tự động đánh số 1, 2, 3...
2. **Không tự thêm tùy chọn "Other" / "Khác"**: Giao diện UI sẽ tự động bổ sung trường nhập liệu tự do ("Other / Nhập câu trả lời khác") cho người dùng.
3. **Tiền tố `(Recommended)`**: Luôn đặt phương án kỹ thuật tốt nhất lên đầu tiên và gắn tiền tố `(Recommended)`.
4. **Không viết "chọn tất cả những gì áp dụng"**: Khi `is_multi_select: true`, UI sẽ tự hiển thị ghi chú này.

---

## 2. Quy Trình Vận Hành `grill-me` Trước Khi Viết Code

Khi nhận bất kỳ yêu cầu mới nào (tính năng mới, sửa lỗi, tái cấu trúc kiến trúc, API/UI):

### Bước 1: Trình Bày Tóm Tắt & Danh Sách File Dự Kiến
Trước khi gọi `ask_question`, Agent phản hồi bản tóm tắt kế hoạch ngắn gọn ra màn hình:
- **Giải pháp đề xuất**: Kiến trúc, luồng xử lý và thư viện dự kiến.
- **Danh sách file tác động**: Ghi rõ các file sẽ tạo mới (`[+]`), chỉnh sửa (`[*]`), hoặc xóa (`[-]`).
- **Tuân thủ quy chuẩn**: Xác nhận kế hoạch đã tính đến `hardened-backend-security` (nếu là backend) hoặc `ui-ux-pro-architect` (nếu là frontend).

### Bước 2: Gọi Tool `ask_question` (Interactive Box)
Tạo từ 1 đến 3 câu hỏi trắc nghiệm cốt lõi để người dùng quyết định:
1. **Câu hỏi Kiến trúc / Tech Choice**: Lựa chọn giải pháp kỹ thuật, cấu trúc schema hoặc thư viện.
2. **Câu hỏi Edge Cases / UX / Security**: Xử lý các tình huống biên nhạy cảm hoặc hành vi trải nghiệm.
3. **Câu hỏi Phê Duyệt Kế Hoạch (Plan Approval Gate)**:
   - Câu hỏi: *"Bạn có phê duyệt kế hoạch triển khai trên để tôi bắt đầu viết code không?"*
   - Options:
     * `(Recommended) Đồng ý, tiến hành viết code theo kế hoạch đề xuất`
     * `Cần điều chỉnh lại giải pháp kỹ thuật`
     * `Chỉ tạo khung sườn (boilerplate/schema) trước để tôi xem xét`

### Bước 3: Dừng Lại Chờ Người Dùng Chọn (Hard Stop)
- Kết thúc lượt gọi tool mà không tự ý sửa code.
- Chờ phản hồi được submit từ Interactive Box của người dùng.
- Khi người dùng gửi kết quả: Đọc lựa chọn của người dùng và thực thi code chính xác theo các tùy chọn đã chốt.

