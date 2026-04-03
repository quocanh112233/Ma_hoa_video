# Prompt 12 – Final Audit: Kiểm tra Toàn bộ Hệ thống

---

## 📝 TL;DR

- **Input:** Toàn bộ codebase sau khi hoàn thành Prompt 01-11
- **Output:** Báo cáo audit chi tiết + danh sách bugs/fixes đã thực hiện
- **Phạm vi:** Security, Crypto correctness, Integration, Code quality, UX consistency
- **Key constraint:** PHẢI chạy test thực tế, KHÔNG chỉ review code tĩnh
- **Definition of Done:** Tất cả 6 checklist pass + 0 lỗi Critical/High + báo cáo hoàn chỉnh

---

## 🎭 Role

Bạn là **Tech Lead & Security Auditor** chịu trách nhiệm cuối cùng cho chất lượng dự án. Nhiệm vụ là thực hiện kiểm tra toàn diện (comprehensive audit) trước khi phê duyệt dự án, bao gồm chạy thử thực tế, kiểm tra bảo mật, đánh giá code quality, và xác nhận UX consistency.

---

## 📋 Context

- Tất cả 11 prompts trước đã được implement
- Dự án có 4 modules crypto: AES, RSA, Hybrid, Signature
- Security baseline từ Prompt 03 cần được verify trên toàn bộ codebase
- Audit này quyết định dự án có đạt chuẩn "production-ready" hay không

---

## 🔗 Dependencies (từ tất cả prompt trước)

- **Prompt 01:** Setup monorepo
- **Prompt 02:** Wireframes (kiểm tra UI có match wireframe)
- **Prompt 03:** Security baseline + Security Checklist 14 điểm
- **Prompt 04-05:** AES Backend + Frontend
- **Prompt 06-07:** RSA Backend + Frontend
- **Prompt 08-09:** Hybrid Backend + Frontend
- **Prompt 10-11:** Signature Backend + Frontend

---

## 🎯 Mục tiêu

Thực hiện 6 nhóm kiểm tra:

1. **Security Audit** — 14 điểm Security Checklist
2. **Crypto Correctness** — Mã hóa/giải mã đúng cho tất cả 4 modules
3. **Integration Test** — End-to-end flow hoạt động
4. **Code Quality** — TypeScript, Python style, import structure
5. **UX Consistency** — Text, layout, responsive
6. **Performance Baseline** — Đo thời gian cho các tệp khác kích thước

---

## ✅ Checklist 1: Security Audit

> Chạy lại toàn bộ Security Checklist 14 điểm từ Prompt 03:

| # | Tiêu chí | Cách kiểm tra | Pass? |
|---|----------|--------------|-------|
| S01 | AES-256-GCM | `grep -rn "CBC\|ECB" app/` → 0 kết quả | ☐ |
| S02 | Nonce unique | Review code: `os.urandom(12)` trong mỗi chunk | ☐ |
| S03 | RSA-OAEP-SHA256 | `grep -rn "PKCS1v15" app/` → 0 kết quả | ☐ |
| S04 | RSA-PSS | Review signature_service.py: dùng PSS | ☐ |
| S05 | os.urandom() | `grep -rn "random" app/ \| grep -v "os.urandom"` → 0 kết quả crypto | ☐ |
| S06 | Không log key | `grep -rn "print.*key\|log.*key" app/` → 0 kết quả | ☐ |
| S07 | Cleanup temp | Review: không có temp file tồn tại sau response | ☐ |
| S08 | File validation | Review: validate_video_file() được gọi ở mọi endpoint upload | ☐ |
| S09 | Rate limiting | Test: gửi 15 requests → request 11+ trả 429 | ☐ |
| S10 | No stack trace | Gây lỗi server → response chỉ có message, không có traceback | ☐ |
| S11 | Security headers | `curl -I /health` → security headers present | ☐ |
| S12 | Key cleared (FE) | DevTools: sau download, key state = "" | ☐ |
| S13 | No localStorage | DevTools → Application → Local Storage → trống | ☐ |
| S14 | Validate trước API | Nhập sai → Zod error ngay, network tab trống | ☐ |

---

## ✅ Checklist 2: Crypto Correctness

> Kiểm tra mỗi module mã hóa/giải mã đúng:

### AES Round-trip Test

```bash
# Bước 1: Mã hóa video test
curl -X POST http://localhost:8000/api/aes/encrypt \
  -F "file=@test_video.mp4" \
  -o test_video.mp4.enc -D enc_headers.txt

# Lấy key từ header
KEY=$(grep "X-Key-B64" enc_headers.txt | cut -d' ' -f2)

# Bước 2: Giải mã
curl -X POST http://localhost:8000/api/aes/decrypt \
  -F "enc_file=@test_video.mp4.enc" \
  -F "key_b64=$KEY" \
  -o decrypted_video.mp4

# Bước 3: So sánh SHA-256
sha256sum test_video.mp4 decrypted_video.mp4
# → 2 hash PHẢI giống nhau
```

### RSA Round-trip Test

```bash
# Tương tự nhưng ≤ 5MB, dùng PEM keys
```

### Hybrid Round-trip Test

```bash
# Tương tự, không giới hạn kích thước
```

### Signature Round-trip Test

```bash
# Sign-encrypt → Decrypt-verify → signature_valid = true
# → Hash match
```

### Tamper Detection Test

```bash
# Sửa 1 byte trong file .enc → decrypt → PHẢI fail
# Sửa 1 byte trong file .signed.enc → verify → PHẢI = false hoặc GCM fail
```

---

## ✅ Checklist 3: Integration Test (End-to-End)

| # | Kịch bản E2E | Hành động | Kết quả |
|---|-------------|-----------|---------|
| E01 | AES encrypt+decrypt qua UI | FE: upload → mã hóa → download .enc → giải mã → download video | Video xem được |
| E02 | RSA keygen+encrypt+decrypt qua UI | FE: sinh key → mã hóa ≤5MB → giải mã | Video xem được |
| E03 | Hybrid qua UI | FE: dùng RSA key → mã hóa video lớn → giải mã | Video xem được |
| E04 | Signature qua UI | FE: 2 cặp key → ký+mã hóa → giải mã+xác minh | ✅ Hợp lệ + video |
| E05 | Cross-page key sharing | Tạo key ở /rsa → dùng ở /hybrid | Key tự điền |
| E06 | Signature invalid | /signature → sai sender key → verify | ❌ Không hợp lệ |
| E07 | File lớn (>500MB) | AES + Hybrid encrypt/decrypt | Streaming hoàn thành |
| E08 | Rate limiting | Gửi 15 requests liên tục | Request thứ 11 → 429 |
| E09 | Sai file type | Upload .txt thay vì video | FE Zod reject ngay |

---

## ✅ Checklist 4: Code Quality

### Backend (Python)

```bash
# Linting
cd backend && pip install flake8
flake8 app/ --max-line-length 120 --count

# Import check: không có unused imports
# Namespace check: không có circular imports
python -c "from app.main import app; print('Import OK')"

# Type hints
grep -rn "def " app/ | grep -v "-> " | head -20
# → Ít kết quả nhất có thể (mọi function nên có return type hint)
```

### Frontend (TypeScript)

```bash
# Type check
cd frontend && npx tsc --noEmit
# → 0 errors

# Build check
npm run build
# → 0 errors, 0 warnings (trừ warning nhỏ)

# No 'any' type
grep -rn ": any" src/ | wc -l
# → Ít nhất có thể (< 5)
```

### Cấu trúc project

- [ ] Không có file duplicate (prompt04/05 bug đã fix)
- [ ] Import paths đúng (không có import từ `../../..` 3+ levels)
- [ ] Mỗi component 1 file
- [ ] Services tách riêng khỏi components
- [ ] Stores (Zustand) tách riêng khỏi components

---

## ✅ Checklist 5: UX Consistency

| # | Tiêu chí | Cách kiểm tra | Pass? |
|---|----------|--------------|-------|
| U01 | 100% tiếng Việt | Đọc toàn bộ UI | ☐ |
| U02 | Navbar 5 links | Kiểm tra tất cả trang | ☐ |
| U03 | Active state | Click từng link trên navbar | ☐ |
| U04 | FileDropZone nhất quán | So sánh drop zone trên 4 trang | ☐ |
| U05 | Progress bar style | So sánh progress bar trên 4 trang | ☐ |
| U06 | Error message style | Gây lỗi → kiểm tra alert/toast style | ☐ |
| U07 | Button color convention | Encrypt = blue, Decrypt = green | ☐ |
| U08 | Responsive mobile | Resize browser ≤ 640px → check 4 trang | ☐ |
| U09 | Tab style nhất quán | So sánh tab UI trên 4 trang | ☐ |
| U10 | Download button style | So sánh trên 4 trang | ☐ |

---

## ✅ Checklist 6: Performance Baseline

| Kịch bản | Kích thước | Thời gian tối đa | Đo thực tế |
|---------|-----------|-----------------|-----------|
| AES encrypt | 10 MB | 5 giây | ___ giây |
| AES encrypt | 100 MB | 15 giây | ___ giây |
| AES encrypt | 1 GB | 60 giây | ___ giây |
| RSA encrypt | 5 MB | 120 giây | ___ giây |
| Hybrid encrypt | 100 MB | 15 giây | ___ giây |
| Signature sign+enc | 100 MB | 20 giây | ___ giây |
| SHA-256 hash | 1 GB | 10 giây | ___ giây |

---

## 📋 Cấu trúc Báo cáo Audit

```markdown
# BÁO CÁO AUDIT - Video Crypto

## 1. Tóm tắt
- Ngày audit: ___
- Phiên bản: 1.0
- Kết quả tổng thể: ✅ PASS / ❌ FAIL

## 2. Security Audit
- Pass: __/14
- Fail: __/14
- Chi tiết: [bảng từ Checklist 1]

## 3. Crypto Correctness
- AES: ✅/❌
- RSA: ✅/❌
- Hybrid: ✅/❌
- Signature: ✅/❌
- Tamper Detection: ✅/❌

## 4. Integration Test
- Pass: __/9
- Chi tiết: [bảng từ Checklist 3]

## 5. Code Quality
- Backend flake8 errors: __
- Frontend TypeScript errors: __
- 'any' usage count: __

## 6. UX Consistency
- Pass: __/10
- Chi tiết: [bảng từ Checklist 5]

## 7. Performance
- [bảng từ Checklist 6]

## 8. Bugs Found & Fixed
| # | Mô tả | Severity | Status |
|---|-------|----------|--------|
| B01 | ... | Critical/High/Medium/Low | Fixed/Open |

## 9. Recommendations
- Danh sách đề xuất cải thiện cho phiên bản tương lai

## 10. Kết luận
- ✅ PASS: Dự án đạt chuẩn
- HOẶC ❌ FAIL: Dự án cần sửa thêm [liệt kê]
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Chạy test THỰC TẾ (curl commands, browser clicks) — KHÔNG chỉ đọc code
- SHA-256 hash comparison cho round-trip tests — KHÔNG chấp nhận "xem được"
- Mọi bug Critical/High PHẢI fix trước khi pass
- Báo cáo phải follow template ở trên

❌ **NGHIÊM CẤM:**
- KHÔNG pass audit nếu còn bug Critical
- KHÔNG skip checklist item nào — đánh dấu N/A nếu không áp dụng
- KHÔNG claim "đã test" mà không có lệnh/ảnh chụp chứng minh

---

## ⚠️ Edge Cases cần test đặc biệt

| # | Edge Case | Cách test |
|---|-----------|----------|
| 1 | File 0 bytes | Upload file rỗng → phải reject |
| 2 | File .exe → .mp4 | Đổi extension → magic bytes reject |
| 3 | Unicode filename | Video tên tiếng Việt dấu → phải xử lý được |
| 4 | Concurrent requests | 5 requests mã hóa song song → tất cả phải thành công |
| 5 | Key whitespace | Key có trailing newlines → phải strip |
| 6 | Stale key (RSA) | Dùng key cũ trên session mới → phải hoạt động |

---

## 🔄 Self-Verification

Sau khi hoàn thành audit, bạn PHẢI:

1. **6/6 checklists** đều có kết quả cụ thể (không để trống)
2. **0 bug Critical** và **0 bug High** mở
3. **Báo cáo** theo đúng template 10 sections
4. **Round-trip** tất cả 4 modules đều pass SHA-256 verification
5. **Kết luận** rõ ràng: PASS hoặc FAIL + lý do

Chỉ báo **"HOÀN THÀNH"** khi toàn bộ checklist pass và có báo cáo hoàn chỉnh.
