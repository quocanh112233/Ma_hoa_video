# Report 08 – Hybrid Backend: Mã hóa Lai

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | POST /api/hybrid/encrypt hoạt động | ⏳ Chưa implement | |
| 2 | AES session key được sinh mới mỗi lần encrypt | ⏳ Chưa implement | |
| 3 | AES session key được RSA encrypt và nhúng vào file | ⏳ Chưa implement | |
| 4 | Video mã hóa bằng AES-256-GCM theo chunks | ⏳ Chưa implement | |
| 5 | Cấu trúc file .hybrid.enc đúng định dạng | ⏳ Chưa implement | |
| 6 | Header JSON có đủ fields (algorithm, rsa_key_size_bits, chunk_size,...) | ⏳ Chưa implement | |
| 7 | POST /api/hybrid/decrypt hoạt động | ⏳ Chưa implement | |
| 8 | Không giới hạn kích thước file | ⏳ Chưa implement | |
| 9 | Decrypt sai receiver key → 400 message rõ ràng | ⏳ Chưa implement | |
| 10 | POST /api/hybrid/generate-session-key hoạt động | ⏳ Chưa implement | |
| 11 | Stream response, không load toàn bộ video vào RAM | ⏳ Chưa implement | |
| 12 | Tái sử dụng aes_service + rsa_service (không copy-paste) | ⏳ Chưa implement | |
| 13 | Pydantic validator cho public/private key PEM | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | POST /api/hybrid/generate-session-key → key 32 bytes | ⏳ | |
| T02 | Encrypt video 10MB + RSA-2048 public key | ⏳ | |
| T03 | Mở .hybrid.enc hex editor → header có "HYBRID-RSA2048-AES256GCM" | ⏳ | |
| T04 | Decrypt đúng receiver key → video gốc xem được | ⏳ | |
| T05 | Decrypt sai key → 400 với message rõ | ⏳ | |
| T06 | Encrypt video >500MB → không timeout, không crash | ⏳ | |
| T07 | Encrypt video ~1 tiếng → hoàn thành | ⏳ | |
| T08 | Decrypt video lớn → video gốc xem được | ⏳ | |
| T09 | Dùng RSA-4096 key → hoạt động, header ghi đúng key_size | ⏳ | |
| T10 | PEM sai → 422 validation error | ⏳ | |
| T11 | So sánh tốc độ encrypt 100MB: Hybrid vs RSA thuần | ⏳ | Hybrid: _s / RSA: N/A (>5MB) |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM cho video (không CBC/ECB) | ✅ | ⏳ | Tầng AES của Hybrid |
| S02 | Nonce mới cho mỗi chunk AES | ✅ | ⏳ | Tái sử dụng từ aes_service |
| S03 | RSA-OAEP-SHA256 để encrypt AES key | ✅ | ⏳ | Tầng RSA của Hybrid |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase Hybrid |
| S05 | os.urandom() cho AES session key | ✅ | ⏳ | Session key mới mỗi lần |
| S06 | AES session key không log ra console/file | ✅ | ⏳ | Key chỉ tồn tại trong memory |
| S07 | Temp file xóa sau xử lý | ✅ | ⏳ | Với file >1GB nếu có dùng temp |
| S08 | File validation: extension + MIME + magic bytes | ✅ | ⏳ | Không giới hạn size |
| S09 | Rate limiter trên endpoint encrypt/decrypt | ✅ | ⏳ | Từ Prompt 03 |
| S10 | Error không expose stack trace | ✅ | ⏳ | Phân biệt lỗi RSA vs AES |
| S11 | Security headers trong response | ✅ | ⏳ | Middleware từ Prompt 03 |
| S12 | Frontend: key cleared sau download | — N/A | — | Thuộc Prompt 09 |
| S13 | Frontend: key không lưu localStorage | — N/A | — | Thuộc Prompt 09 |
| S14 | Frontend: validate trước khi gọi API | — N/A | — | Thuộc Prompt 09 |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 09 (Hybrid Frontend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
