# Prompt 02 – Wireframe & UI Design (HTML tĩnh)

---

## 📝 TL;DR

- **Input:** Cấu trúc dự án từ Prompt 01
- **Output:** 5 file HTML tĩnh (wireframe) + 1 file components + 1 file guide
- **Files to create:** `wireframes/index.html`, `aes.html`, `rsa.html`, `hybrid.html`, `signature.html`, `components.html`, `wireframe-guide.md`
- **Key constraint:** Chỉ HTML/CSS tĩnh (Tailwind CDN), KHÔNG có JavaScript logic
- **Definition of Done:** Mở browser thấy layout đúng, tab switching hoạt động, responsive, toàn text tiếng Việt

---

## 🎭 Role

Bạn là **UI/UX Designer** chuyên về wireframing cho ứng dụng web. Nhiệm vụ là tạo bộ wireframe HTML tĩnh hoàn chỉnh cho toàn bộ 5 trang của dự án Video Crypto, đảm bảo layout rõ ràng, trực quan, và phù hợp để developer frontend có thể implement React components chính xác.

---

## 📋 Context

- Prompt 01 đã hoàn thành: Monorepo skeleton đang chạy
- Wireframe là bản thiết kế visual TRƯỚC KHI viết React code
- Wireframe dùng Tailwind CSS CDN (không cần build)
- Mục đích: Developer và stakeholder review layout trước khi code

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** Sử dụng cấu trúc 5 trang đã định nghĩa (Home, AES, RSA, Hybrid, Signature)
- **KHÔNG thay đổi:** Không sửa bất kỳ file nào trong `backend/` hoặc `frontend/src/`

---

## 🎯 Mục tiêu

Tạo 7 file trong folder `wireframes/`:

1. **index.html** — Trang chủ: 4 card điều hướng + bảng so sánh
2. **aes.html** — 2 tab: Mã hóa / Giải mã
3. **rsa.html** — 3 tab: Sinh Key / Mã hóa / Giải mã + banner cảnh báo
4. **hybrid.html** — 3 tab: Quy trình / Mã hóa / Giải mã
5. **signature.html** — 4 tab: Tổng quan / Ký & Mã hóa / Giải mã & Xác minh / Chỉ ký
6. **components.html** — Library 7 components dùng chung
7. **wireframe-guide.md** — Hướng dẫn ký hiệu và màu sắc

---

## 📁 File cần tạo

```
wireframes/
├── index.html          ← Trang chủ
├── aes.html            ← Trang AES
├── rsa.html            ← Trang RSA
├── hybrid.html         ← Trang Hybrid
├── signature.html      ← Trang Chữ ký số
├── components.html     ← Component library
└── wireframe-guide.md  ← Hướng dẫn
```

---

## ✅ Yêu cầu chức năng chi tiết

### Shared Layout (áp dụng cho TẤT CẢ trang)

```
┌──────────────────────────────────────────────┐
│  🔐 Video Crypto    [AES] [RSA] [Lai] [CKS]  │  ← Header + Navbar
├──────────────────────────────────────────────┤
│                                              │
│              (Nội dung trang)                 │
│                                              │
├──────────────────────────────────────────────┤
│  © 2024 Video Crypto - Đồ án mã hóa video    │  ← Footer
└──────────────────────────────────────────────┘
```

### 1. `index.html` — Trang chủ

```
┌──────────────────────────────────────────────┐
│           🔐 Mã hóa & Giải mã Video          │
│        Bảo vệ video với mật mã học hiện đại   │
├──────────────────────────────────────────────┤
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐     │
│  │ AES  │  │ RSA  │  │ Lai  │  │ CKS  │     │  ← 4 cards
│  │ 256  │  │ 2048 │  │Hybrid│  │ Sig  │     │
│  │      │  │      │  │      │  │      │     │
│  │[Bắt  │  │[Bắt  │  │[Bắt  │  │[Bắt  │     │
│  │ đầu] │  │ đầu] │  │ đầu] │  │ đầu] │     │
│  └──────┘  └──────┘  └──────┘  └──────┘     │
├──────────────────────────────────────────────┤
│        Bảng so sánh 4 phương pháp             │
│  ┌──────────────────────────────────────┐    │
│  │       │ AES │ RSA │ Hybrid │ CKS    │    │
│  │ Tốc độ│ ⚡  │ 🐢  │  ⚡   │  ⚡   │    │
│  │ Kích  │ ∞   │ 5MB │  ∞    │  ∞    │    │
│  │ Bảo mật│ Cao │ Cao │ Rất cao│RấtCao │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

- 4 cards điều hướng: mỗi card có icon, tên phương pháp, mô tả ngắn, nút "Bắt đầu →"
- Bảng so sánh: 4 cột, các tiêu chí (Tốc độ, Kích thước file, Bảo mật, Ứng dụng)
- Mỗi card link đến trang tương ứng

### 2. `aes.html` — Trang AES

```
┌──────────────────────────────────────────────┐
│           Mã hóa AES-256-GCM                 │
│   [🔒 Mã hóa]  [🔓 Giải mã]                │  ← 2 tabs
├──────────────────────────────────────────────┤
│  Tab Mã hóa:                                 │
│  ┌────────────────────────────┐              │
│  │   📁 Kéo thả video vào đây │              │  ← FileDropZone
│  │   hoặc click để chọn file  │              │
│  └────────────────────────────┘              │
│  Tên file: ________  Kích thước: ___        │
│                                              │
│  🔑 Key:  ◉ Tạo tự động  ○ Nhập thủ công    │  ← Radio
│  [    Input key base64 (readonly)     ] [📋] │
│                                              │
│  [████████████████░░░░░░] 75%  Đang mã hóa..│  ← Progress
│                                              │
│  ┌─ Kết quả ─────────────────────────┐      │
│  │ ⬇️ Tải file .enc    ⬇️ Tải file .key│      │
│  │ ⚠️ Hãy lưu file .key cẩn thận!    │      │  ← Warning
│  │ [ Mã hóa video khác ]             │      │
│  └────────────────────────────────────┘      │
└──────────────────────────────────────────────┘
```

- Tab Mã hóa: FileDropZone + radio key mode + progress bar + download links
- Tab Giải mã: FileDropZone (.enc) + input key + progress + download video

### 3. `rsa.html` — Trang RSA

```
┌──────────────────────────────────────────────┐
│  ⚠️ Cảnh báo học thuật: RSA chỉ hỗ trợ     │  ← Banner amber
│     file ≤ 5MB. Chỉ dùng cho mục đích       │
│     học tập và so sánh.                      │
├──────────────────────────────────────────────┤
│  [🔑 Sinh Key]  [🔒 Mã hóa]  [🔓 Giải mã]  │  ← 3 tabs
├──────────────────────────────────────────────┤
│  Tab Sinh Key:                               │
│  Key size: [RSA-2048 ▾]                      │
│  [ Tạo cặp Key RSA ]                        │
│                                              │
│  Public Key:  [textarea readonly] [📋][⬇️]  │
│  Private Key: [████ BLUR ████████] [👁️][📋][⬇️]│  ← Blur
│  🔴 KHÔNG chia sẻ Private Key!              │
│  [ Kiểm tra cặp key ] → ✅ / ❌              │
├──────────────────────────────────────────────┤
│  Tab Mã hóa:                                │
│  ⏱️ Thời gian: 00:45.2 giây                 │  ← Timer
│  💡 So sánh: AES mã hóa file này chỉ <1s   │  ← Hint
└──────────────────────────────────────────────┘
```

- Banner amber cảnh báo luôn hiển thị ở đầu trang (không thể đóng)
- Tab Sinh Key: select key size + generate button + PEM display + blur private key
- Tab Mã hóa: FileDropZone ≤5MB + timer khi mã hóa
- Tab Giải mã: upload .rsa.enc + private key PEM

### 4. `hybrid.html` — Trang Hybrid

```
┌──────────────────────────────────────────────┐
│    Mã hóa Lai (Hybrid Encryption)            │
│    ✅ Không giới hạn kích thước file          │  ← Badge xanh lá
│  [📊 Quy trình]  [🔒 Mã hóa]  [🔓 Giải mã] │
├──────────────────────────────────────────────┤
│  Tab Quy trình (mặc định):                  │
│                                              │
│  Sơ đồ Mã hóa:                              │
│  [Video] →AES-GCM→ [Video mã hóa]  (xanh)  │
│  [AES Key] →RSA PK→ [Enc AES Key]  (tím)    │
│  [Video mã hóa]+[Enc Key] = [.hybrid.enc]   │ (xanh lá)
│                                              │
│  Bảng so sánh RSA thuần vs Hybrid:           │
│  ┌──────────────────────────────────┐        │
│  │        │ RSA thuần  │ Hybrid    │        │
│  │ File   │ ≤ 5MB     │ Không hạn │        │
│  │ Tốc độ │ Rất chậm  │ Nhanh    │        │
│  │ Ứng dụng│ Học thuật │ HTTPS    │        │
│  └──────────────────────────────────┘        │
├──────────────────────────────────────────────┤
│  Tab Giải mã: metadata card sau upload       │
│  ┌─ Thông tin file ────────────────┐        │
│  │ Tên gốc: video.mp4             │        │
│  │ Kích thước: 52.4 MB            │        │
│  │ Thuật toán: HYBRID-RSA2048-AES │        │
│  │ Ngày tạo: 2024-01-01           │        │
│  └─────────────────────────────────┘        │
└──────────────────────────────────────────────┘
```

- Tab "Quy trình" là mặc định khi mở trang
- Sơ đồ có 3 màu: xanh dương (AES), tím (RSA), xanh lá (kết quả)
- Bảng so sánh RSA vs Hybrid

### 5. `signature.html` — Trang Chữ ký số

```
┌──────────────────────────────────────────────┐
│    Chữ ký số (Digital Signature)             │
│  [🔒 Bảo mật] [🛡️ Toàn vẹn] [✍️ Xác thực]  │  ← 3 badges
│  [📊 Tổng quan] [✍️ Ký] [🔓 Giải mã] [🔏 Chỉ ký]│
├──────────────────────────────────────────────┤
│  Tab Ký & Mã hóa:                           │
│  ┌── Sender ──────┐  ┌── Receiver ─────┐    │  ← 2 cột
│  │ Private Key:   │  │ Public Key:     │    │
│  │ [Upload .pem]  │  │ [Upload .pem]  │    │
│  │ hoặc dán text  │  │ hoặc dán text  │    │
│  └────────────────┘  └────────────────┘    │
│                                              │
│  Timeline:                                   │
│  ✅ Tính SHA-256 hash video                  │
│  🔄 Ký bằng RSA-PSS...                      │
│  ⏸️ Mã hóa AES key bằng RSA                 │
│  ⏸️ Mã hóa video bằng AES-GCM               │
├──────────────────────────────────────────────┤
│  Tab Giải mã & Xác minh:                    │
│  ┌─ Kết quả xác minh ────────────────┐      │
│  │ ✅ Chữ ký HỢP LỆ                  │      │  ← Xanh lá
│  │ SHA-256 Expected: abc123...        │      │
│  │ SHA-256 Actual:   abc123...        │      │
│  │ [⬇️ Tải video gốc]               │      │
│  └────────────────────────────────────┘      │
│  HOẶC:                                       │
│  ┌─ Kết quả ────────────────────────┐        │
│  │ ❌ Chữ ký KHÔNG HỢP LỆ           │       │  ← Đỏ
│  │ ⚠️ Video có thể đã bị giả mạo    │       │
│  │ [⬇️ Tải video (cảnh báo)]        │       │
│  └────────────────────────────────────┘      │
└──────────────────────────────────────────────┘
```

- 3 badges tính chất ở đầu trang
- Layout 2 cột cho Sender/Receiver keys
- SignatureVerifyResult: card xanh (hợp lệ) hoặc đỏ (không hợp lệ)
- Hiển thị SHA-256 Expected vs Actual

### 6. `components.html` — Component Library

Hiển thị 7 components với đầy đủ states:

| Component | States cần thể hiện |
|-----------|-------------------|
| FileDropZone | idle, dragover, selected, error |
| ProgressBar | 0%, 50%, 100%, error |
| AlertBox | info (blue), success (green), warning (amber), error (red) |
| TabSwitcher | 2 tabs, 3 tabs, 4 tabs |
| KeyInput | empty, filled, error, blur/hidden |
| DownloadButton | idle, loading, ready |
| StatusTimeline | pending, active (spinner), done, error |

### 7. `wireframe-guide.md`

- Bảng ký hiệu: `[___]` = input, `( )` = radio, `[x]` = checkbox
- Bảng màu: blue-600 (mã hóa), green-600 (giải mã), amber-500 (cảnh báo), red-600 (lỗi)
- Quy tắc responsive: mobile-first, stack columns trên mobile

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Dùng Tailwind CSS CDN `<script src="https://cdn.tailwindcss.com"></script>`
- Tab switching phải hoạt động bằng CSS/JS đơn giản (onclick toggle class)
- Sections phải responsive (stack trên mobile, grid trên desktop)
- Toàn bộ text phải bằng tiếng Việt
- Shared header + footer giống nhau ở tất cả trang

❌ **NGHIÊM CẤM:**
- KHÔNG dùng React, Vue, hoặc bất kỳ framework JS nào
- KHÔNG gọi API backend
- KHÔNG implement logic crypto thực sự
- KHÔNG dùng hình ảnh/icon từ bên ngoài (chỉ dùng emoji Unicode)

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG tạo wireframe bằng tool bên ngoài (Figma, Sketch) — phải viết HTML
2. KHÔNG copy HTML từ template bên ngoài — viết từ đầu theo spec
3. KHÔNG dùng `display: none` hardcode — tab switching phải dùng JS toggle
4. KHÔNG dùng pixel units cho spacing — dùng Tailwind classes (`p-4`, `m-2`)

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Layout trang chủ | Mở `index.html` | 4 card + bảng so sánh hiển thị đúng |
| T02 | Tab AES | Mở `aes.html`, click tab chuyển | Tab chuyển mượt, nội dung đúng |
| T03 | FileDropZone | Xem component | Vùng kéo thả rõ ràng, border dashed |
| T04 | Cảnh báo AES | Xem tab Mã hóa sau kết quả | Có box cảnh báo vàng |
| T05 | Banner RSA | Mở `rsa.html` | Banner amber ở đầu trang |
| T06 | Private key blur | Xem Tab Sinh Key | Private key bị blur, không đọc được |
| T07 | Tab quy trình Hybrid | Mở `hybrid.html` | Tab "Quy trình" active mặc định |
| T08 | Metadata card | Xem tab Giải mã hybrid | Card thông tin file rõ ràng |
| T09 | 3 badges Signature | Mở `signature.html` | Thấy 3 badges tính chất |
| T10 | Layout 2 cột | Xem tab Ký & Mã hóa | Sender bên trái, Receiver bên phải |
| T11 | Verify result card | Xem mẫu kết quả xác minh | Card xanh (hợp lệ) và đỏ (không hợp lệ) |
| T12 | Components | Mở `components.html` | 7 components với đầy đủ states |
| T13 | Responsive | Thu nhỏ browser width ≤ 640px | Layout stack, không vỡ |
| T14 | Tiếng Việt | Đọc toàn bộ text | 100% tiếng Việt |

---

## 📋 Checklist tự kiểm tra

- [ ] File `index.html` có 4 card điều hướng + bảng so sánh
- [ ] File `aes.html` có 2 tab, FileDropZone, progress bar
- [ ] File `rsa.html` có banner amber + 3 tab + private key blur
- [ ] File `hybrid.html` có sơ đồ màu sắc + bảng so sánh
- [ ] File `signature.html` có 3 badges + layout 2 cột + verify result card
- [ ] File `components.html` có đủ 7 components với states
- [ ] File `wireframe-guide.md` có bảng ký hiệu + bảng màu
- [ ] Shared Header + Navbar + Footer giống nhau ở tất cả trang
- [ ] Tab switching hoạt động bằng click
- [ ] Responsive: không vỡ layout trên mobile (≤ 640px)
- [ ] 100% text tiếng Việt
