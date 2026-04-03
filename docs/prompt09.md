# Prompt 09 – Hybrid Frontend: React UI Mã hóa Lai

---

## 📝 TL;DR

- **Input:** Backend Hybrid API từ Prompt 08 đang hoạt động
- **Output:** Trang `/hybrid` hoàn chỉnh với 3 tab: Quy trình / Mã hóa / Giải mã
- **Files to create:** `HybridPage.tsx`, `HybridDiagram.tsx`, `HybridEncryptPanel.tsx`, `HybridDecryptPanel.tsx`, `MetadataCard.tsx`, `useHybridStore.ts`, `hybridSchema.ts`, `hybridService.ts`
- **Key constraint:** Tab "Quy trình" là mặc định, sơ đồ 3 màu, metadata card khi giải mã
- **Definition of Done:** End-to-end encrypt/decrypt hoạt động + 14 test cases pass

---

## 🎭 Role

Bạn là **Frontend Engineer** chuyên về React. Nhiệm vụ là xây dựng giao diện trang **Mã hóa Lai (Hybrid)**, nơi giáo dục người dùng về quy trình Hybrid Encryption (giống HTTPS thực tế), đồng thời cung cấp chức năng mã hóa/giải mã video kích thước lớn.

---

## 📋 Context

- Phase 3 Frontend — Prompt 08 (Hybrid Backend) đã hoàn thành
- Hybrid = AES mã hóa video + RSA mã hóa AES key (giống TLS handshake)
- Không giới hạn kích thước file (khác với trang RSA thuần)
- Cần visual diagram giải thích quy trình cho mục đích giáo dục
- Route: `/hybrid`

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** React Router route `/hybrid` → `HybridPage.tsx`
- **Từ Prompt 03:** Import `downloadBlob`, `clearSensitiveData`
- **Từ Prompt 05:** Tái sử dụng `FileDropZone`, `ProgressBar` (nếu tách riêng)
- **Từ Prompt 07:** Import `useRSAStore` để đọc key (nút "Dùng key từ trang RSA")
- **Từ Prompt 08:** API endpoints: `/api/hybrid/encrypt`, `/api/hybrid/decrypt`
- **KHÔNG thay đổi:** Không sửa bất kỳ file AES/RSA frontend nào

---

## 🎯 Mục tiêu

Xây dựng trang `/hybrid` gồm 3 tab:

1. **Tab Quy trình** (mặc định) — sơ đồ trực quan giải thích Hybrid Encryption
2. **Tab Mã hóa** — upload video + RSA public key → download `.hybrid.enc`
3. **Tab Giải mã** — upload `.hybrid.enc` + RSA private key → metadata card + download video

---

## 📁 File cần tạo/chỉnh sửa

```
frontend/src/
├── pages/HybridPage.tsx                       ← Implement đầy đủ
├── components/crypto/
│   ├── HybridDiagram.tsx                      ← Tạo mới
│   ├── HybridEncryptPanel.tsx                 ← Tạo mới
│   ├── HybridDecryptPanel.tsx                 ← Tạo mới
│   └── MetadataCard.tsx                       ← Tạo mới (REUSABLE)
├── stores/useHybridStore.ts                   ← Tạo mới
├── schemas/hybridSchema.ts                    ← Tạo mới
└── services/hybridService.ts                  ← Tạo mới
```

---

## ✅ Yêu cầu chức năng

### HybridPage.tsx

- Tiêu đề: "Mã hóa Lai (Hybrid Encryption)"
- Badge xanh lá: "✅ Không giới hạn kích thước file"
- 3 tab: "📊 Quy trình" (active), "🔒 Mã hóa", "🔓 Giải mã"

### Tab Quy trình (HybridDiagram.tsx)

**Sơ đồ mã hóa (3 màu):**

```
┌──────────────────────────────────────────────────────┐
│ SƠ ĐỒ MÃ HÓA LAI                                   │
│                                                      │
│  ┌───────┐   AES-GCM    ┌─────────────┐             │
│  │ Video │──────────────→│ Video mã hóa│  (blue)     │
│  └───────┘      ↑        └─────────────┘             │
│            ┌─────────┐                               │
│            │ AES Key │ (random, 1 lần dùng)          │
│            └─────────┘                               │
│                │ RSA Public Key                       │
│                ↓                                     │
│         ┌──────────────┐                             │
│         │ AES Key đã   │  (purple)                   │
│         │ mã hóa (RSA) │                             │
│         └──────────────┘                             │
│                │                                     │
│                ↓                                     │
│  ┌──────────────────────────────────┐                │
│  │         .hybrid.enc             │  (green)        │
│  │ = Header + Encrypted AES Key   │                 │
│  │   + Video chunks mã hóa AES    │                 │
│  └──────────────────────────────────┘                │
└──────────────────────────────────────────────────────┘
```

- Mã màu: **Xanh dương (blue)** = luồng AES, **Tím (purple)** = luồng RSA, **Xanh lá (green)** = kết quả
- Sơ đồ dùng CSS/HTML, KHÔNG dùng ảnh PNG/SVG bên ngoài

**Bảng so sánh RSA thuần vs Hybrid:**

| Tiêu chí | RSA thuần | Hybrid (RSA + AES) |
|---------|-----------|-------------------|
| Kích thước file | ≤ 5MB | Không giới hạn |
| Tốc độ | Rất chậm | Nhanh (AES) |
| Bảo mật | Cao | Rất cao |
| Ứng dụng thực tế | Học thuật | HTTPS, Email mã hóa |

### Tab Mã hóa (HybridEncryptPanel)

- **FileDropZone:** nhận video, KHÔNG có giới hạn kích thước
- **RSA Public Key:** 3 cách cung cấp:
  1. "📁 Upload file .pem"
  2. "📋 Dán key text"
  3. "🔗 Dùng key từ trang RSA" (đọc từ `useRSAStore.publicKeyPem`)
  - Nếu useRSAStore chưa có key → disable option 3, tooltip "Chưa tạo key ở trang RSA"
- Nút "🔒 Mã hóa Hybrid"
- Progress bar + status text: "Đang mã hóa video bằng AES-GCM... → Đang mã hóa AES key bằng RSA..."
- Kết quả: download `.hybrid.enc`, metadata summary

### Tab Giải mã (HybridDecryptPanel)

- **FileDropZone:** nhận `.hybrid.enc`
- **Sau khi upload:** parse metadata từ header → hiển thị `MetadataCard`:

  ```
  ┌─ Thông tin file ──────────────────────────┐
  │ 📄 Tên gốc:       video.mp4              │
  │ 📦 Kích thước gốc: 52.4 MB               │
  │ 🔐 Thuật toán:     HYBRID-RSA2048-AES256GCM │
  │ 📊 Số chunks:      5                     │
  │ 📅 Ngày tạo:       2024-01-01            │
  └───────────────────────────────────────────┘
  ```

- **RSA Private Key:** Radio upload/paste (giống RSA page)
- Nút "🔓 Giải mã" → progress → download video gốc

### MetadataCard.tsx (REUSABLE)

```typescript
interface MetadataCardProps {
  metadata: {
    original_filename: string
    original_size: number
    algorithm: string
    chunk_count: number
    created_at: string
    [key: string]: any
  }
}
```

- Render table key-value
- Format file size (MB/GB)
- Format date (locale VN)
- Tái sử dụng ở trang Signature (Prompt 11)

---

## 🔧 Yêu cầu kỹ thuật

### `stores/useHybridStore.ts`

```typescript
interface HybridStore {
  // Encrypt
  encryptFile: File | null
  publicKeyPem: string
  publicKeySource: 'upload' | 'paste' | 'rsa-store'
  encryptProgress: number
  encryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  encryptedBlob: Blob | null
  encryptError: string | null

  // Decrypt
  decryptEncFile: File | null
  metadata: Record<string, any> | null  // parsed from .hybrid.enc header
  privateKeyPem: string
  decryptProgress: number
  decryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  decryptedBlob: Blob | null
  decryptedFilename: string | null
  decryptError: string | null
}
```

### Parse metadata client-side

```typescript
// Đọc 4 bytes đầu → headerLen → đọc header JSON → hiện MetadataCard
// CHỈ đọc metadata, KHÔNG decrypt video phía frontend
async function parseEncFileMetadata(file: File): Promise<Record<string, any>> {
  const buffer = await file.slice(0, 4).arrayBuffer();
  const headerLen = new DataView(buffer).getUint32(0);
  const headerBuffer = await file.slice(4, 4 + headerLen).arrayBuffer();
  const headerJson = new TextDecoder().decode(headerBuffer);
  return JSON.parse(headerJson);
}
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Tab "Quy trình" phải là tab mặc định khi mở trang
- Sơ đồ 3 màu là CSS/HTML, không dùng ảnh ngoài
- MetadataCard PHẢI parse metadata từ file header ngay khi upload (client-side)
- Nút "Dùng key từ trang RSA" đọc từ Zustand store (cross-page)

❌ **NGHIÊM CẤM:**
- KHÔNG decrypt video phía frontend — gửi toàn bộ lên backend
- KHÔNG dùng `window.crypto` cho AES — backend xử lý crypto
- KHÔNG giới hạn file size ở frontend (khác RSA page)

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý frontend |
|-----------|-------------------|
| useRSAStore chưa có key | Disable nút "Dùng key từ RSA", hiện tooltip |
| File `.hybrid.enc` bị truncated | Parse metadata sẽ fail → alert "File không hợp lệ" |
| Metadata JSON malformed | try/catch → alert lỗi |
| Video > 1GB upload | Progress bar phải hoạt động, Axios timeout 5 phút |

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Mở trang | Navigate `/hybrid` | Tab "Quy trình" active, sơ đồ hiện |
| T02 | Sơ đồ 3 màu | Xem visual | 3 màu rõ ràng: blue, purple, green |
| T03 | Bảng so sánh | Xem dưới sơ đồ | Bảng RSA vs Hybrid |
| T04 | Badge xanh | Xem tiêu đề | "✅ Không giới hạn kích thước" |
| T05 | Upload video | Tab Mã hóa + file 50MB | FileDropZone chấp nhận |
| T06 | Key từ RSA | Tạo key ở `/rsa` → quay lại `/hybrid` | "Dùng key từ RSA" hoạt động |
| T07 | Key paste | Paste PEM → encrypt | Hoạt động |
| T08 | Encrypt | Full flow → download `.hybrid.enc` | File download thành công |
| T09 | Parse metadata | Upload `.hybrid.enc` ở tab Giải mã | MetadataCard hiện thông tin |
| T10 | Metadata fields | Kiểm tra MetadataCard | Tên file, kích thước, thuật toán, ngày tạo |
| T11 | Decrypt đúng | Upload + private key đúng → giải mã | Video gốc download, xem được |
| T12 | Decrypt sai | Upload + sai key | Alert đỏ lỗi |
| T13 | Video lớn | Upload 500MB+ | Không crash, progress bar hoạt động |
| T14 | Tab switching | Chuyển 3 tab | Mượt, state giữ nguyên trong session |

---

## 📋 Checklist tự kiểm tra

- [ ] Tab "Quy trình" là mặc định
- [ ] Sơ đồ 3 màu hiển thị đúng
- [ ] Bảng so sánh RSA vs Hybrid
- [ ] Badge "Không giới hạn kích thước"
- [ ] "Dùng key từ RSA" hoạt động (cross-store)
- [ ] MetadataCard parse header client-side
- [ ] MetadataCard hiển thị đủ fields
- [ ] Encrypt/Decrypt round-trip hoạt động
- [ ] Sai key → alert rõ ràng
- [ ] Toàn bộ text tiếng Việt
- [ ] TypeScript không lỗi
