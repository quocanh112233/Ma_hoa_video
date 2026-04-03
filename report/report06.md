# Report 06 – RSA Backend: FastAPI Encrypt/Decrypt

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | POST /api/rsa/generate-keypair hoạt động | ⏳ Chưa implement | |
| 2 | Hỗ trợ RSA-2048 và RSA-4096 | ⏳ Chưa implement | |
| 3 | Từ chối key_size ngoài 2048/4096 → 422 | ⏳ Chưa implement | |
| 4 | POST /api/rsa/encrypt hoạt động (≤5MB) | ⏳ Chưa implement | |
| 5 | Dùng OAEP-SHA256 padding (không phải PKCS1v15) | ⏳ Chưa implement | |
| 6 | Giới hạn 5MB, trả 413 nếu vượt | ⏳ Chưa implement | |
| 7 | Cấu trúc file .rsa.enc đúng định dạng | ⏳ Chưa implement | |
| 8 | Header JSON có field "warning" | ⏳ Chưa implement | |
| 9 | Response header X-Warning có trong response | ⏳ Chưa implement | |
| 10 | POST /api/rsa/decrypt hoạt động | ⏳ Chưa implement | |
| 11 | Decrypt sai key → 400 với message rõ | ⏳ Chưa implement | |
| 12 | POST /api/rsa/validate-keypair hoạt động | ⏳ Chưa implement | |
| 13 | validate-keypair dùng encrypt/decrypt thực sự để kiểm tra | ⏳ Chưa implement | |
| 14 | Pydantic validator cho PEM public/private key | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Generate RSA-2048 → JSON có PEM keys | ⏳ | |
| T02 | Generate RSA-4096 → thành công (chậm hơn) | ⏳ | |
| T03 | key_size = 1024 → 422 validation error | ⏳ | |
| T04 | validate-keypair đúng → `{"valid": true}` | ⏳ | |
| T05 | validate-keypair sai (mix 2 cặp) → `{"valid": false}` | ⏳ | |
| T06 | Encrypt file ~1MB + public key → download .rsa.enc | ⏳ | |
| T07 | Decrypt đúng key → file gốc, nội dung khớp | ⏳ | |
| T08 | Decrypt sai key → 400 | ⏳ | |
| T09 | File 10MB → 413 "File vượt quá 5MB" | ⏳ | |
| T10 | PEM sai format → 422 | ⏳ | |
| T11 | Inspect response → có header X-Warning | ⏳ | |
| T12 | Đo thời gian encrypt file 5MB | ⏳ | Ghi lại: ___ giây |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM (không CBC/ECB) | — N/A | — | Phase này dùng RSA thuần |
| S02 | Nonce mới cho mỗi chunk | — N/A | — | RSA không dùng nonce |
| S03 | RSA-OAEP-SHA256 padding (không PKCS1v15) | ✅ | ⏳ | Core của phase này |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase RSA |
| S05 | os.urandom() cho key generation | ✅ | ⏳ | Dùng cryptography lib sinh key |
| S06 | Key không log ra console/file | ✅ | ⏳ | Private key đặc biệt quan trọng |
| S07 | Temp file xóa sau xử lý | ✅ | ⏳ | Nếu có dùng temp file |
| S08 | File validation: extension + MIME + magic bytes | ✅ | ⏳ | Giới hạn 5MB + validate type |
| S09 | Rate limiter được apply trên endpoint | ✅ | ⏳ | Từ Prompt 03 |
| S10 | Error không expose stack trace | ✅ | ⏳ | Test T08 |
| S11 | Security headers có trong response | ✅ | ⏳ | Middleware từ Prompt 03 |
| S12 | Frontend: key cleared sau download | — N/A | — | Thuộc Prompt 07 |
| S13 | Frontend: key không lưu localStorage | — N/A | — | Thuộc Prompt 07 |
| S14 | Frontend: validate trước khi gọi API | — N/A | — | Thuộc Prompt 07 |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 07 (RSA Frontend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
