---
name: ui-ux-pro-architect
description: "Bộ quy chuẩn và kỹ năng kiến trúc giao diện UI/UX chuyên nghiệp: hoàn thiện 4 trạng thái UI (Loading, Empty, Error, Data), chống double-submit và phản hồi tức thì, tuân thủ Design System (spacing 4px/8px, typography hierarchy, WCAG AA, tokens), tối ưu Mobile-first, Semantic HTML và hỗ trợ điều hướng bàn phím chuẩn a11y."
---

# UI/UX Pro Architect Standard

Bộ kỹ năng và quy chuẩn kiến trúc giao diện người dùng (UI) và trải nghiệm người dùng (UX) chuyên nghiệp bắt buộc, đảm bảo mọi giao diện ứng dụng web/mobile đều đạt độ hoàn thiện cao, trực quan, nhất quán với hệ thống thiết kế (Design System) và đáp ứng các tiêu chuẩn tiếp cận phổ quát (Accessibility - a11y).

---

## 1. Complete 4 UI States Matrix (Bắt Buộc Đủ 4 Trạng Thái Giao Diện)

Mọi component, trang (page) hoặc container có tải dữ liệu từ API hoặc xử lý bất đồng bộ **bắt buộc** phải hiện thực đầy đủ cả 4 trạng thái giao diện:

```
                  ┌──────────────────────┐
                  │ 1. Loading (Tải)     │
                  └──────────┬───────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
┌──────────────────────┐          ┌──────────────────────┐
│ 2. Empty (Rỗng)      │          │ 3. Error (Lỗi)       │
└──────────────────────┘          └──────────────────────┘
            │                                 │
            └────────────────┬────────────────┘
                             ▼
                  ┌──────────────────────┐
                  │ 4. Data / Success    │
                  └──────────────────────┘
```

### 1.1. Loading State (Đang Tải)
- **Skeleton Screen**: Ưu tiên Skeleton loader có hình khối mô phỏng bố cục chính xác của dữ liệu sắp xuất hiện (kích thước avatar, độ dài dòng text, khung card) thay vì spinner đơn độc toàn trang.
- **Tránh Layout Shift**: Thiết lập kích thước cố định hoặc `min-height` để tránh hiện tượng giật giật (Cumulative Layout Shift - CLS) khi dữ liệu nạp xong.
- **Micro-loading**: Khi chỉ cập nhật một phần nhỏ (ví dụ toggle switch, load more), hiển thị inline spinner hoặc shimmer tinh tế mà không khóa toàn bộ màn hình.

### 1.2. Empty State (Trạng Thái Rỗng)
- Không bao giờ để lại một khoảng trắng trống trơn, khó hiểu.
- **3 yếu tố bắt buộc của Empty State**:
  1. *Hình ảnh / Biểu tượng trực quan* (Illustration hoặc Icon ngữ cảnh).
  2. *Thông điệp rõ ràng*: Giải thích vì sao chưa có dữ liệu (ví dụ: "Chưa có dự án nào được tạo").
  3. *Call to Action (CTA)*: Nút hành động trực tiếp hướng dẫn người dùng bước tiếp theo (ví dụ: nút "+ Tạo dự án đầu tiên").

### 1.3. Error State (Trạng Thái Lỗi)
- **User-friendly copy**: Diễn giải lỗi bằng ngôn ngữ tự nhiên, tích cực và dễ hiểu đối với người dùng cuối; tuyệt đối không hiển thị mã lỗi thô như `Error 500: Internal Server Error` hoặc exception message.
- **Nút Thử Lại (Retry Action)**: Luôn đi kèm nút "Thử lại" (`Retry`) hoặc "Làm mới trang" cho phép người dùng kích hoạt lại request mà không cần F5 toàn bộ trang.
- **Giữ lại ngữ cảnh**: Nếu lỗi xảy ra khi submit form, không xóa sạch các trường dữ liệu người dùng đã nhập.

### 1.4. Data / Success State (Hiển Thị Dữ Liệu Đầy Đủ)
- Xử lý các trường dữ liệu rỗng/nullable một cách lịch thiệp (fallback placeholder như `—`, `Chưa cập nhật`, `N/A`).
- Tránh lỗi text tràn khung bằng cách cấu hình `truncate`, `line-clamp` kèm `title` tooltip nếu cần hiển thị đầy đủ.

---

## 2. Anti Double-Submit & Clear Feedback (Chống Bấm Trùng Lặp & Phản Hồi Tức Thì)

### 2.1. Kiểm Soát Trạng Thái Nút Bấm Khi Submit Form / Action
- Ngay khi người dùng kích hoạt hành động gửi yêu cầu (Submit, Lưu, Thanh toán, Xóa):
  - Nút bấm phải lập tức chuyển sang trạng thái `disabled` và kích hoạt cờ `isSubmitting`/`isLoading`.
  - Hiển thị spinner thu nhỏ hoặc icon đang quay kèm text trạng thái (ví dụ: `Đang lưu...`, `Đang xử lý...`).
  - Vô hiệu hóa các nút phụ liên quan để ngăn chặn việc người dùng click nhiều lần (Double-submit prevention) gây duplicate requests.

```tsx
// ✅ Mẫu chuẩn nút bấm chống Double-Submit
<button
  type="submit"
  disabled={isSubmitting || !isValid}
  className="relative inline-flex items-center justify-center px-4 py-2 text-sm font-medium transition-colors rounded-lg bg-primary text-primary-foreground hover:bg-primary/90 disabled:opacity-50 disabled:cursor-not-allowed"
>
  {isSubmitting && <Spinner className="w-4 h-4 mr-2 animate-spin" />}
  {isSubmitting ? 'Đang lưu thay đổi...' : 'Lưu thay đổi'}
</button>
```

### 2.2. Phản Hồi Trực Quan Rõ Ràng (Feedback Loop)
- **Toast Notifications**: Hiển thị thông báo Toast nổi ở góc màn hình khi hành động thành công hoặc thất bại (`Thành công: Đã lưu hồ sơ`, `Lỗi: Vui lòng kiểm tra lại kết nối mạng`). Toast phải tự ẩn sau 3 - 5 giây và có nút đóng `x`.
- **Inline Alert / Field Errors**: Lỗi nhập liệu phải hiển thị ngay dưới trường input tương ứng với màu cảnh báo rõ ràng và icon báo hiệu.
- **Hành động nguy hiểm**: Các thao tác xóa vĩnh viễn, hủy hợp đồng bắt buộc phải qua Modal/Dialog xác nhận 2 bước với nút hành động mang phong cách cảnh báo (`destructive`/đỏ).

---

## 3. Design System Consistency (Nhất Quán Hệ Thống Thiết Kế)

### 3.1. Hệ Thống Spacing 4px / 8px Grid
- Mọi khoảng cách (margin, padding, gap) phải tuân thủ nghiêm ngặt bội số 4px/8px:
  - `4px` (0.25rem - xs / compact spacing)
  - `8px` (0.5rem - sm)
  - `12px` (0.75rem - md-tight)
  - `16px` (1rem - standard base)
  - `24px` (1.5rem - lg)
  - `32px` (2rem - xl)
  - `48px` (3rem - 2xl)
- Cấm tùy tiện dùng các giá trị lẻ như `margin: 13px`, `padding: 7px`.

### 3.2. Phân Cấp Typography (Visual Hierarchy)
- Xác định rõ vai trò của từng cấp bậc văn bản:
  - **H1 (Page Title)**: 28px - 36px, `font-bold` hoặc `font-semibold`, line-height tight.
  - **H2 (Section Header)**: 20px - 24px, `font-semibold`.
  - **H3 (Card / Modal Header)**: 16px - 18px, `font-medium`.
  - **Body (Default)**: 14px - 16px, `font-normal`, line-height relaxed (1.5 - 1.6) để dễ đọc.
  - **Caption / Meta**: 12px - 13px, `text-muted-foreground`.

### 3.3. WCAG AA Color Contrast & Design Tokens
- **Độ tương phản màu**:
  - Tối thiểu **4.5:1** đối với văn bản thông thường (body text, caption).
  - Tối thiểu **3:1** đối với văn bản cỡ lớn (18pt / 24px trở lên) và các thành phần giao diện đồ họa (viền input, button active).
- **Sử dụng Token thay vì Hardcode Hex Code**:
  - Không hardcode trực tiếp mã màu `#1E293B`, `#3B82F6` trong file component.
  - Luôn sử dụng biến CSS / Design Tokens / Lớp Tailwind ngữ nghĩa (`bg-background`, `text-foreground`, `bg-primary`, `text-muted-foreground`, `border-border`) để đảm bảo hỗ trợ Dark Mode và tính tùy biến toàn hệ thống.

---

## 4. Accessibility (a11y) & Mobile-First Responsive (Dễ Tiếp Cận & Tương Thích Thiết Bị)

### 4.1. Mobile-First & Touch Target Size
- Thiết kế bố cục ưu tiên hiển thị tối ưu trên thiết bị di động trước (`320px` - `480px`), sau đó mở rộng dần sang Tablet (`768px`) và Desktop (`1024px+`).
- Mọi phần tử tương tác cảm ứng (nút, icon button, tab, checkbox) phải có kích thước vùng chạm tối thiểu là **44 x 44px** (hoặc padding tương đương) để người dùng chạm chính xác trên màn hình cảm ứng.

### 4.2. Semantic HTML Chuẩn Mực
- Sử dụng đúng thẻ ngữ nghĩa theo tiêu chuẩn HTML5:
  - Dùng `<button>` cho hành động (trigger modal, submit form). Cấm dùng `<div onClick>` hoặc `<span onClick>`.
  - Dùng `<a>` (hoặc Link component) cho việc chuyển hướng URL.
  - Dùng `<main>`, `<nav>`, `<header>`, `<footer>`, `<aside>`, `<section>` để phân định các vùng trang.
  - Các input luôn gắn kèm `<label htmlFor="...">` tương ứng để đầu đọc màn hình (Screen Readers) có thể nhận diện.

### 4.3. Keyboard Navigation & Focus Indicators
- Mọi tính năng trên giao diện phải có thể thao tác hoàn toàn bằng bàn phím (`Tab`, `Shift+Tab`, `Enter`, `Space`, `Arrow Keys`).
- **Focus Ring rõ ràng**: Không bao giờ ẩn viền focus (`outline: none`) mà không có trạng thái thay thế; luôn kích hoạt `focus-visible:ring-2 focus-visible:ring-offset-2`.
- **Modal / Dialog Focus Trap**:
  - Khi mở Modal, tiêu điểm bàn phím phải tự động chuyển vào phần tử đầu tiên của Modal và bị giữ bên trong (focus trap).
  - Nhấn phím `Esc` phải lập tức đóng Modal / Popover.
  - Khi đóng Modal, tiêu điểm phải hoàn trả về nút bấm đã mở Modal trước đó.

---

## Checklist Kiểm Định Giao Diện Trước Khi Bàn Giao

1. [ ] Component bất đồng bộ đã xử lý đủ 4 trạng thái: Loading (Skeleton), Empty (có CTA), Error (có Retry), Data?
2. [ ] Form và action button đã có trạng thái `isSubmitting`/`disabled` kèm loading spinner để chống bấm nhiều lần chưa?
3. [ ] Có thông báo Toast/Inline alert rõ ràng sau khi thực hiện thao tác chưa?
4. [ ] Bố cục khoảng cách đã tuân thủ hệ thống lưới 4px/8px chưa?
5. [ ] Màu sắc văn bản có đạt độ tương phản WCAG AA (>= 4.5:1) và sử dụng biến/tokens ngữ nghĩa không?
6. [ ] Giao diện có co giãn mượt mà trên Mobile (Mobile-first, touch target >= 44px) không?
7. [ ] Có dùng đúng thẻ Semantic HTML (`<button>`, `<main>`, `<label>`) và hỗ trợ thao tác bàn phím (Focus ring, phím Esc) chưa?

