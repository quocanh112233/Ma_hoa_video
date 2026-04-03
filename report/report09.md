# Report 09 – Hybrid Frontend: React UI Mã hóa Lai

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | Badge "✅ Không giới hạn kích thước file" | ⏳ Chưa implement | |
| 2 | 3 tab: "📊 Quy trình" (mặc định) / "🔒 Mã hóa" / "🔓 Giải mã" | ⏳ Chưa implement | |
| 3 | Tab Quy trình: sơ đồ mã hóa + sơ đồ giải mã có màu sắc | ⏳ Chưa implement | |
| 4 | Bảng so sánh RSA thuần vs Hybrid trong Tab Quy trình | ⏳ Chưa implement | |
| 5 | Nút "🔑 Dùng key từ trang RSA" (tích hợp useRSAStore) | ⏳ Chưa implement | |
| 6 | Parse metadata từ .hybrid.enc trên client-side | ⏳ Chưa implement | |
| 7 | Metadata card hiển thị sau khi upload .hybrid.enc | ⏳ Chưa implement | |
| 8 | Timeline trạng thái mã hóa (Sinh AES key / Encrypt AES key / Encrypt video) | ⏳ Chưa implement | |
| 9 | Không giới hạn kích thước file upload | ⏳ Chưa implement | |
| 10 | Giải mã sai key → alert đỏ rõ ràng | ⏳ Chưa implement | |
| 11 | Zustand store (useHybridStore) đủ state theo spec | ⏳ Chưa implement | |
| 12 | Zod validate file .hybrid.enc (HybridEncFileSchema) | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Mở /hybrid → Tab "Quy trình" active mặc định | ⏳ | |
| T02 | Sơ đồ có màu sắc (xanh/tím/lá), bảng so sánh | ⏳ | |
| T03 | Click "Dùng key từ trang RSA" → public key tự điền | ⏳ | |
| T04 | Mã hóa video 10MB → download .hybrid.enc | ⏳ | |
| T05 | Mã hóa video >100MB → không bị giới hạn | ⏳ | |
| T06 | Timeline hiển thị các bước trong khi mã hóa | ⏳ | |
| T07 | Upload .hybrid.enc → Metadata card hiện (tên, kích thước, ngày, algo) | ⏳ | |
| T08 | Giải mã đúng receiver key → video gốc xem được | ⏳ | |
| T09 | Giải mã sai key → alert đỏ | ⏳ | |
| T10 | Upload file .enc (không phải .hybrid.enc) → Zod error | ⏳ | |
| T11 | Mã hóa video ~1 tiếng → thành công | ⏳ | |
| T12 | Decrypt video lớn → video gốc xem được | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM | — N/A | — | Crypto ở backend (Prompt 08) |
| S02 | Nonce mới cho mỗi chunk | — N/A | — | Crypto ở backend |
| S03 | RSA-OAEP-SHA256 | — N/A | — | Crypto ở backend |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase Hybrid |
| S05 | os.urandom() cho key generation | — N/A | — | Ở backend |
| S06 | Key không log ra console/file | ✅ | ⏳ | Không log RSA key vào console |
| S07 | Temp file xóa sau xử lý | — N/A | — | Không có temp file ở frontend |
| S08 | File validation: extension .hybrid.enc | ✅ | ⏳ | Zod HybridEncFileSchema |
| S09 | Rate limiter | — N/A | — | Ở backend |
| S10 | Error không expose stack trace | ✅ | ⏳ | Hiện message tiếng Việt |
| S11 | Security headers | — N/A | — | Ở backend |
| S12 | Frontend: key cleared sau download | ✅ | ⏳ | Xóa RSA key khỏi store sau dùng |
| S13 | Frontend: key không lưu localStorage | ✅ | ⏳ | Kiểm tra DevTools → Application |
| S14 | Frontend: validate PEM trước khi gọi API | ✅ | ⏳ | Tái dùng PEMPublicKeySchema từ Prompt 07 |

---

## 🔍 Phân tích

### Items ngoài yêu cầu

-

### Items thiếu sót

-

### Items vi phạm

-

---

## 📝 Ghi chú kỹ thuật

---

## ✅ Kết luận

- [ ] **ĐẠT** – Chuyển sang Prompt 10 (Chữ ký số Backend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
