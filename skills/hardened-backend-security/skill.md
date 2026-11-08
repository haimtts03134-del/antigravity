---
name: hardened-backend-security
description: "Bộ quy chuẩn và kỹ năng kiến trúc backend bảo mật cao: cấm shortcut/TODO giữ chỗ, bắt buộc validate schema chặt chẽ, chống Mass Assignment, triệt tiêu BOLA/IDOR bằng query scoping, bọc Transaction/Concurrency/Idempotency cho nghiệp vụ nhạy cảm, lọc output qua DTO, bắt lỗi an toàn không rò rỉ stack trace, parameterized query và phân trang giới hạn limit chống DoS."
---

# Hardened Backend Security Standard

Bộ kỹ năng và quy chuẩn kỹ thuật bắt buộc nhằm đảm bảo mọi hệ thống Backend, API, Database và xử lý dữ liệu đạt chuẩn an ninh phòng vệ chiều sâu (Defense-in-Depth), toàn vẹn dữ liệu và chống chịu các vector tấn công phổ biến theo OWASP API Security Top 10.

---

## 1. Zero Shortcuts & Zero Placeholders (Tuyệt Đối Không Viết Tắt)

- **Cấm hoàn toàn các comment giữ chỗ**: Tuyệt đối không sử dụng `// TODO: validate input`, `// TODO: add authentication later`, `// TODO: handle transaction`, `/* check permission here */` hoặc giả định "sẽ xử lý sau".
- **Code mẫu phải an toàn hoàn chỉnh**: Mọi hàm, controller, service hay endpoint khi được viết ra phải có đầy đủ lớp kiểm tra tính hợp lệ, xác thực danh tính, phân quyền và bọc xử lý lỗi ngay từ dòng đầu tiên. Không rút ngắn code bằng cách bỏ qua các bước kiểm tra an toàn.

---

## 2. Strict Input Validation & Anti-Mass Assignment (Kiểm Soát Dữ Liệu Đầu Vào Chặt Chẽ)

### 2.1. Bắt Buộc Validate Toàn Bộ Input bằng Schema
Mọi điểm tiếp nhận dữ liệu từ bên ngoài (`req.body`, `req.query`, `req.params`, `headers`) đều bắt buộc phải được định nghĩa và kiểm định qua Schema Parser (Zod, Pydantic, Joi, class-validator) trước khi chuyển vào tầng Service/Database.

- **Strict Type Checking**: Kiểm tra kiểu dữ liệu, định dạng (UUID, email, regex chuẩn), độ dài tối thiểu/tối đa, dải giá trị số hợp lệ.
- **Reject Unexpected Properties**: Kích hoạt chế độ loại bỏ hoặc từ chối các trường lạ (Zod: `.strict()` / `.strip()`; Pydantic: `extra = "forbid"`).

### 2.2. Chống Mass Assignment (Overposting)
- **Tuyệt đối cấm** truyền trực tiếp toàn bộ payload request vào ORM hoặc Database:
  ```typescript
  // ❌ NGUY HIỂM (Mass Assignment): Cho phép hacker ghi đè role, balance, is_admin
  await prisma.user.update({ where: { id }, data: req.body });
  ```
- **Chuẩn an toàn bắt buộc**: Luôn định nghĩa DTO/Schema cho từng hành động cụ thể và chỉ trích xuất chính xác các trường được phép cập nhật (Whitelist):
  ```typescript
  // ✅ AN TOÀN: Chỉ trích xuất các trường được phép thay đổi
  const { displayName, bio, avatarUrl } = updateProfileSchema.parse(req.body);
  await prisma.user.update({
    where: { id: currentUserId },
    data: { displayName, bio, avatarUrl }
  });
  ```

---

## 3. BOLA / IDOR Protection (Chống Đọc/Sửa/Xóa Trái Phép Tài Nguyên Theo ID)

Lỗ hổng Broken Object Level Authorization (BOLA/IDOR) là nguyên nhân hàng đầu gây rò rỉ và can thiệp trái phép dữ liệu.

### 3.1. Xác Thực Quyền Sở Hữu Tại Tầng Truy Vấn Dữ Liệu (Query-Level Scoping)
- Không chỉ dựa vào middleware xác thực người dùng đã đăng nhập (`authMiddleware`).
- Mọi câu lệnh SQL/ORM thực hiện đọc (`find`), cập nhật (`update`), hoặc xóa (`delete`) theo ID bắt buộc phải gắn kèm điều kiện ràng buộc quyền sở hữu (`user_id`, `tenant_id`, hoặc tổ chức của caller):
  ```typescript
  // ❌ NGUY HIỂM (BOLA/IDOR): Bất kỳ ai có ID tài liệu đều có thể sửa/xóa
  await prisma.document.update({
    where: { id: documentId },
    data: { title }
  });

  // ✅ AN TOÀN (Query Scoping): Ràng buộc ID tài liệu VÀ quyền sở hữu của caller
  const result = await prisma.document.updateMany({
    where: {
      id: documentId,
      ownerId: currentUser.id,
      tenantId: currentUser.tenantId, // Bắt buộc cho mô hình Multi-tenant
      isDeleted: false
    },
    data: { title, updatedAt: new Date() }
  });

  if (result.count === 0) {
    throw new NotFoundException("Tài nguyên không tồn tại hoặc bạn không có quyền thao tác");
  }
  ```

### 3.2. Không Phân Biệt Lỗi Khi Truy Cập Trái Phép
- Trả về mã lỗi thống nhất `404 Not Found` (hoặc `403 Forbidden` tuỳ chính sách) để tránh rò rỉ sự tồn tại của ID tài nguyên thuộc về người khác (Enumeration Defense).

---

## 4. Database Transaction, Concurrency & Idempotency (Giao Dịch & Đảm Bảo Tính Toàn Vẹn)

### 4.1. Bắt Buộc Transaction Cho Nghiệp Vụ Nhạy Cảm
Mọi luồng xử lý liên quan đến:
- Tài chính, biến động số dư, ví điện tử.
- Trừ kho sản phẩm, đặt phòng, vé giữ chỗ.
- Cập nhật đồng thời từ 2 bảng/collection trở lên.
**BẮT BUỘC** phải được bọc trong Database Transaction với Isolation Level phù hợp (`READ COMMITTED` hoặc `SERIALIZABLE`).

### 4.2. Xử Lý Race Condition & Concurrency Control
- **Atomic Operation**: Sử dụng câu lệnh nguyên tử tại DB thay vì đọc lên application rồi ghi đè:
  ```sql
  -- ✅ An toàn: Trừ kho trực tiếp kèm điều kiện kiểm tra số lượng
  UPDATE products
  SET stock = stock - :quantity, updated_at = NOW()
  WHERE id = :productId AND stock >= :quantity;
  ```
- **Row-Level Locking**: Sử dụng `SELECT ... FOR UPDATE` khi cần đọc và tính toán phức tạp trước khi ghi trong cùng transaction.
- **Optimistic Locking**: Bổ sung cột `version` cho các bảng tài nguyên có tần suất sửa đổi đồng thời cao (`UPDATE table SET val = :val, version = version + 1 WHERE id = :id AND version = :currentVersion`).

### 4.3. Đảm Bảo Tính Bất Khả Biến (Idempotency)
- Với các API POST/PATCH xử lý thanh toán, chuyển tiền hoặc tạo đơn hàng, yêu cầu header `Idempotency-Key` từ client.
- Kiểm tra khoá idempotency tại distributed cache/DB (Redis hoặc DB table có unique index). Nếu request trùng đang xử lý hoặc đã hoàn tất, trả về kết quả đã lưu thay vì thực thi lại nghiệp vụ.

---

## 5. Safe Error Handling & Output Sanitization (Bảo Vệ Dữ Liệu Đầu Ra & Xử Lý Lỗi Tập Trung)

### 5.1. DTO / Serializer Cho Dữ Liệu Phản Hồi
- Tuyệt đối cấm trả thẳng Model từ ORM/Database về client.
- Bắt buộc ánh xạ qua Response DTO hoặc Serializer để loại bỏ:
  - `password_hash`, `salt`, `mfa_secret`.
  - Token nội bộ, khóa giải mã, refresh tokens.
  - Cấu trúc khóa ngoại nội bộ, cờ kiểm duyệt hệ thống không dành cho người dùng.

### 5.2. Bắt Lỗi Tập Trung & Không Lộ Thông Tin Nhạy Cảm
- Triển khai Global Exception Handler / Error Filter tập trung.
- **Phía Server**: Ghi log chi tiết (stack trace, câu truy vấn SQL kèm request_id) vào hệ thống log bảo mật nội bộ, che mờ (masking/redaction) thông tin định danh cá nhân (PII, thẻ tín dụng).
- **Phía Client**: Chỉ trả về định dạng JSON chuẩn với mã lỗi tường minh và thông điệp an toàn, **tuyệt đối cấm trả stack trace, cú pháp lỗi SQL, tên bảng/cột DB hoặc đường dẫn file hệ thống**.
  ```json
  {
    "success": false,
    "error": {
      "code": "PAYMENT_FAILED",
      "message": "Không thể xử lý giao dịch. Vui lòng thử lại sau.",
      "requestId": "req_01h4k8m9v2z5x"
    }
  }
  ```

---

## 6. SQL/NoSQL Injection Defense & DoS Prevention via Pagination (Phòng Vệ Injection & Chống DoS)

### 6.1. Tham Số Hóa Truy Vấn 100% (Parameterized Queries)
- **Tuyệt đối cấm** nối chuỗi ký tự (`+`, template literals `${...}`) để tạo câu lệnh SQL hoặc MongoDB query.
- Luôn sử dụng Parameterized Query chuẩn (`$1, $2` hoặc `:param`) hoặc các ORM có hỗ trợ Prepared Statements tự động.
- Với NoSQL (MongoDB/Mongoose), cấm truyền trực tiếp `req.body` vào bộ lọc query để tránh toán tử injection (`$gt`, `$ne`, `$where`).

### 6.2. Phân Trang An Toàn Chống Cạn Kiệt Tài Nguyên (DoS)
- Mọi endpoint trả về danh sách bắt buộc phải có phân trang với `limit` tối đa được giới hạn cứng (ví dụ: tối đa 50 hoặc 100 bản ghi/lần).
- Không bao giờ cho phép query không giới hạn (`SELECT * FROM records`):
  ```typescript
  // Giới hạn limit an toàn
  const page = Math.max(1, Number(req.query.page) || 1);
  const requestedLimit = Number(req.query.limit) || 20;
  const safeLimit = Math.min(Math.max(1, requestedLimit), 100); // Tối đa 100
  ```
- Khuyến khích sử dụng Keyset/Cursor-based Pagination đối với các bảng dữ liệu lớn thay vì Offset-based để tối ưu hiệu năng IO và CPU của Database.

---

## Checklist Kiểm Định Trước Khi Hoàn Tất Code Backend

1. [ ] Đã kiểm tra không còn bất kỳ comment `TODO`, `FIXME`, hoặc bỏ sót bước bảo mật nào chưa?
2. [ ] Dữ liệu đầu vào (body, query, params) đã được validate chặt chẽ bằng schema chưa? Đã chống Mass Assignment chưa?
3. [ ] Mọi truy vấn đọc/sửa/xóa theo ID đã được bọc `user_id` / `tenant_id` ngay tại câu query DB chưa?
4. [ ] Nghiệp vụ nhạy cảm nhiều bước đã có Transaction, xử lý Race condition và Idempotency chưa?
5. [ ] Dữ liệu trả về đã lọc sạch password hash, token, secret qua DTO chưa?
6. [ ] Bộ bắt lỗi có đảm bảo không làm lộ raw SQL error hoặc stack trace ra phía client không?
7. [ ] Truy vấn danh sách đã có giới hạn `limit` tối đa và parameterized query chuẩn chưa?

