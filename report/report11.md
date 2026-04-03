# Report 11 – Chữ ký số Frontend: Digital Signature + Hybrid UI

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | 3 badge tính chất: 🔒 Bảo mật / 🛡️ Toàn vẹn / ✍️ Xác thực | ⏳ Chưa implement | |
| 2 | 4 tab: "📊 Tổng quan" / "✍️ Ký & Mã hóa" / "🔓 Giải mã & Xác minh" / "🔏 Chỉ ký/Xác minh" | ⏳ Chưa implement | |
| 3 | Tab Tổng quan: sơ đồ end-to-end Sender → Receiver | ⏳ Chưa implement | |
| 4 | Bảng giải thích 3 tính chất bảo mật (Confidentiality/Integrity/Authentication) | ⏳ Chưa implement | |
| 5 | Tab Ký & Mã hóa: layout 2 cột (Sender key / Receiver key) | ⏳ Chưa implement | |
| 6 | Timeline animated: Tính hash / Ký / Encrypt AES key / Encrypt video | ⏳ Chưa implement | |
| 7 | SHA-256 hash hiển thị sau khi ký xong | ⏳ Chưa implement | |
| 8 | SignatureVerifyResult component: xanh lá (hợp lệ) / đỏ (không hợp lệ) | ⏳ Chưa implement | |
| 9 | Hiển thị cả 2 SHA-256 (Expected vs Actual) để so sánh | ⏳ Chưa implement | |
| 10 | Video vẫn download được dù chữ ký invalid (kèm cảnh báo) | ⏳ Chưa implement | |
| 11 | Parse metadata từ .signed.enc trên client-side | ⏳ Chưa implement | |
| 12 | Tab "Chỉ ký/Xác minh" hoạt động độc lập | ⏳ Chưa implement | |
| 13 | Tái sử dụng Zod schema từ Phase 2 và 3 (không duplicate) | ⏳ Chưa implement | |
| 14 | Đọc verify result từ response headers (X-Signature-Valid,...) | ⏳ Chưa implement | |
| 15 | Toàn bộ text tiếng Việt | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Mở /signature → thấy 3 badge + Tab Tổng quan active | ⏳ | |
| T02 | Tab Tổng quan: sơ đồ đầy đủ, bảng 3 tính chất | ⏳ | |
| T03 | Ký & Mã hóa video 50MB + 2 cặp key khác nhau | ⏳ | |
| T04 | Timeline hiển thị từng bước trong khi xử lý | ⏳ | |
| T05 | SHA-256 hash hiển thị sau khi ký & mã hóa xong | ⏳ | |
| T06 | Upload .signed.enc → Metadata card hiện đầy đủ | ⏳ | |
| T07 | Giải mã & Verify đúng → SignatureVerifyResult màu xanh ✅ | ⏳ | |
| T08 | 2 SHA-256 hash (Expected vs Actual) khớp nhau | ⏳ | |
| T09 | Sai sender public key → đỏ ❌, vẫn có nút download video | ⏳ | |
| T10 | Sai receiver key → 400, không có download, hiện lỗi | ⏳ | |
| T11 | Tab "Chỉ ký" → ký video → download .sig.json | ⏳ | |
| T12 | Tab "Chỉ xác minh" đúng → SignatureVerifyResult xanh | ⏳ | |
| T13 | Tab "Chỉ xác minh" sai (video đã sửa) → đỏ ❌ | ⏳ | |
| T14 | Ký file >500MB → thành công, không crash | ⏳ | |
| T15 | Quy trình end-to-end: sinh key ở /rsa → ký+mã hóa ở /signature → giải mã+xác minh | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM | — N/A | — | Crypto ở backend (Prompt 10) |
| S02 | Nonce mới cho mỗi chunk | — N/A | — | Crypto ở backend |
| S03 | RSA-OAEP-SHA256 padding | — N/A | — | Crypto ở backend |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Crypto ở backend (Prompt 10) |
| S05 | os.urandom() | — N/A | — | Ở backend |
| S06 | Sender Private Key không log ra console/file | ✅ | ⏳ | **Đặc biệt quan trọng** |
| S07 | Temp file xóa sau xử lý | — N/A | — | Không có temp file ở frontend |
| S08 | File validation: .signed.enc, .sig.json | ✅ | ⏳ | Zod SignedEncFileSchema, SigJsonFileSchema |
| S09 | Rate limiter | — N/A | — | Ở backend |
| S10 | Error không expose stack trace | ✅ | ⏳ | Hiện message tiếng Việt |
| S11 | Security headers | — N/A | — | Ở backend |
| S12 | Frontend: Sender Private Key cleared sau khi dùng | ✅ | ⏳ | clearSensitiveData() sau sign |
| S13 | Frontend: key không lưu localStorage | ✅ | ⏳ | Kiểm tra DevTools → Application |
| S14 | Frontend: validate PEM trước khi gọi API | ✅ | ⏳ | Tái dùng schema từ Prompt 07, 09 |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 12 (Final Audit)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
