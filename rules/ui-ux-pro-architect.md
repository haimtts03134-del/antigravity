---
trigger: always_on
description: "Bắt buộc tuân thủ 100% tiêu chuẩn UI/UX Pro Architect khi viết hoặc sửa code Frontend, UI Component, giao diện người dùng."
---

# UI/UX Pro Architect Enforcement Rule

## Quy Tắc Bắt Buộc Khi Viết/Sửa Code Frontend & UI Component

Bất kỳ khi nào Agent thực hiện viết mới, tái cấu trúc hoặc chỉnh sửa code liên quan đến Frontend, Giao diện người dùng, Web/Mobile Component hoặc Form tương tác, **BẮT BUỘC** phải tuân thủ 100% tiêu chuẩn trong skill `ui-ux-pro-architect`:

1. **Hiện Thực Đầy Đủ 4 Trạng Thái UI (Complete 4 UI States)**:
   - Mọi component có lấy dữ liệu bất đồng bộ hoặc gọi API phải có đủ 4 trạng thái:
     * **Loading**: Dùng Skeleton screen (hoặc spinner) mô phỏng cấu trúc, không gây nhảy layout (CLS).
     * **Empty**: Thiết kế trực quan có hình ảnh/icon, thông điệp rõ ràng và nút Call-to-Action (CTA) hướng dẫn tạo mới/bước tiếp theo.
     * **Error**: Thông điệp lỗi thân thiện (tránh biệt ngữ kỹ thuật) kèm nút Thử lại (Retry button).
     * **Data / Success**: Hiển thị dữ liệu hoàn chỉnh, xử lý fallback lịch sự cho dữ liệu rỗng (`—`, `N/A`).

2. **Chống Bấm Trùng Lặp (Anti Double-Submit) & Phản Hồi Tức Thì**:
   - Mọi form submit hoặc action button phải có trạng thái `isSubmitting`/`disabled` kèm loading indicator khi đang gọi API.
   - Luôn phản hồi tức thì kết quả bằng Toast notification hoặc Inline alert (Thành công / Thất bại).
   - Thao tác phá huỷ (Delete/Hủy) phải qua Modal xác nhận 2 bước.

3. **Nhất Quán Hệ Thống Thiết Kế (Design System Consistency)**:
   - Tuân thủ nghiêm ngặt hệ thống spacing bội số 4px/8px (4, 8, 12, 16, 24, 32, 48px).
   - Phân cấp Typography rõ ràng (H1, H2, H3, Body, Caption).
   - Đảm bảo độ tương phản màu đạt chuẩn WCAG AA (tối thiểu 4.5:1 cho text thường, 3:1 cho UI/large text).
   - Luôn dùng biến màu / Design tokens / Semantic Tailwind classes thay vì hardcode mã màu hex/rgb.

4. **Khả Năng Tiếp Cận (Accessibility) & Tương Thích Mobile-First**:
   - Bố cục thích ứng chuẩn Mobile-first, kích thước vùng chạm (touch target) tối thiểu 44x44px.
   - Dùng đúng thẻ Semantic HTML (`<button>`, `<main>`, `<nav>`, `<label htmlFor="...">`). Cấm dùng `<div onClick>`.
   - Hỗ trợ thao tác bàn phím chuẩn: viền `focus-visible` rõ nét, bẫy tiêu điểm (focus trap) và phím `Esc` đóng Modal/Dialog, phím `Enter` submit form.

