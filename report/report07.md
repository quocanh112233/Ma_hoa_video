# Report 07 – RSA Frontend: React UI Mã hóa/Giải mã RSA

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | Banner cảnh báo học thuật màu amber luôn hiển thị | ⏳ Chưa implement | |
| 2 | 3 tab: "🔑 Sinh Key" / "🔒 Mã hóa" / "🔓 Giải mã" | ⏳ Chưa implement | |
| 3 | Tab Sinh Key: select RSA-2048 / RSA-4096 | ⏳ Chưa implement | |
| 4 | Private key blur mặc định + nút toggle hiện/ẩn | ⏳ Chưa implement | |
| 5 | Nút download 2 file PEM riêng (public + private) | ⏳ Chưa implement | |
| 6 | Nút "Kiểm tra cặp key" gọi /api/rsa/validate-keypair | ⏳ Chưa implement | |
| 7 | File >5MB bị block ngay từ frontend (Zod VideoFileSmallSchema) | ⏳ Chưa implement | |
| 8 | Timer đếm giây realtime khi đang mã hóa | ⏳ Chưa implement | |
| 9 | Hiển thị thời gian sau mã hóa xong ("Hoàn thành trong Xs") | ⏳ Chưa implement | |
| 10 | Hint so sánh tốc độ với AES sau khi mã hóa xong | ⏳ Chưa implement | |
| 11 | PEMKeyDisplay component: Copy + Download .pem | ⏳ Chưa implement | |
| 12 | Cảnh báo đỏ "🔴 KHÔNG chia sẻ Private Key!" | ⏳ Chưa implement | |
| 13 | Zustand store (useRSAStore) đủ state theo spec | ⏳ Chưa implement | |
| 14 | Zod validation PEM keys (PEMPublicKeySchema, PEMPrivateKeySchema) | ⏳ Chưa implement | |
| 15 | Tab Giải mã hoạt động đầy đủ | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Mở /rsa → thấy banner amber ngay | ⏳ | |
| T02 | Sinh RSA-2048 → xuất hiện public + private key | ⏳ | |
| T03 | Private key mặc định bị blur | ⏳ | |
| T04 | Click "👁️ Hiện" → private key hiện rõ | ⏳ | |
| T05 | Copy public key → clipboard có nội dung PEM | ⏳ | |
| T06 | Download public.pem + private.pem | ⏳ | |
| T07 | Validate keypair đúng → hiện ✅ "Cặp key hợp lệ" | ⏳ | |
| T08 | Validate keypair sai → hiện ❌ "Cặp key không khớp" | ⏳ | |
| T09 | Upload video >5MB → cảnh báo ngay, không upload | ⏳ | |
| T10 | Mã hóa file ~1MB → timer đếm giây | ⏳ | |
| T11 | Sau mã hóa → thấy "Hoàn thành trong Xs" | ⏳ | |
| T12 | Hint so sánh với AES hiện sau khi xong | ⏳ | |
| T13 | Giải mã đúng key → download file gốc | ⏳ | |
| T14 | Giải mã sai key → alert đỏ lỗi | ⏳ | |
| T15 | Nhập PEM text bừa → Zod validation error | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM (không CBC/ECB) | — N/A | — | Crypto ở backend (Prompt 06) |
| S02 | Nonce mới cho mỗi chunk | — N/A | — | Crypto ở backend |
| S03 | RSA-OAEP-SHA256 padding | — N/A | — | Crypto ở backend (Prompt 06) |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase RSA |
| S05 | os.urandom() cho key generation | — N/A | — | Ở backend (Prompt 06) |
| S06 | Key không log ra console/file | ✅ | ⏳ | Đặc biệt: không log private key |
| S07 | Temp file xóa sau xử lý | — N/A | — | Không có temp file ở frontend |
| S08 | File validation: extension + size ≤5MB | ✅ | ⏳ | Zod VideoFileSmallSchema |
| S09 | Rate limiter được apply | — N/A | — | Ở backend (Prompt 03) |
| S10 | Error không expose stack trace | ✅ | ⏳ | Hiện message tiếng Việt |
| S11 | Security headers | — N/A | — | Ở backend (Prompt 03) |
| S12 | Frontend: key cleared sau download | ✅ | ⏳ | Private key xóa state sau download |
| S13 | Frontend: key không lưu localStorage | ✅ | ⏳ | Kiểm tra DevTools → Application |
| S14 | Frontend: validate PEM trước khi gọi API | ✅ | ⏳ | Zod PEMPublicKeySchema / PEMPrivateKeySchema |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 08 (Hybrid Backend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
