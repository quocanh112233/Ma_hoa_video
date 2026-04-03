# Prompt 11 – Signature Frontend: React UI Chữ ký số

---

## 📝 TL;DR

- **Input:** Backend Signature API từ Prompt 10 đang hoạt động
- **Output:** Trang `/signature` hoàn chỉnh với 4 tab: Tổng quan / Ký & Mã hóa / Giải mã & Xác minh / Chỉ ký
- **Files to create:** `SignaturePage.tsx`, `SignatureDiagram.tsx`, `SignEncryptPanel.tsx`, `DecryptVerifyPanel.tsx`, `SignOnlyPanel.tsx`, `SignatureVerifyResult.tsx`, `useSignatureStore.ts`, `signatureSchema.ts`, `signatureService.ts`
- **Key constraint:** 2 cặp key (Sender + Receiver), timeline hiển thị bước, kết quả xác minh có 2 trạng thái (xanh/đỏ)
- **Definition of Done:** End-to-end flow hoạt động + 16 test cases pass

---

## 🎭 Role

Bạn là **Frontend Engineer** chuyên về React. Nhiệm vụ là xây dựng giao diện trang **Chữ ký số**, giáo dục người dùng về quy trình ký + mã hóa kết hợp, đồng thời cung cấp UI nhập 2 cặp key (Sender/Receiver), hiển thị timeline xử lý, và kết quả xác minh chữ ký rõ ràng.

---

## 📋 Context

- Phase 4 Frontend — Prompt 10 (Signature Backend) đã hoàn thành
- Chữ ký số cần 2 cặp key: Sender (ký) và Receiver (mã hóa)
- Kết quả xác minh: ✅ Hợp lệ (xanh) hoặc ❌ Không hợp lệ (đỏ)
- Tính năng "Chỉ ký" (sign-only) cho phép ký mà không mã hóa
- Route: `/signature`
- Đây là trang phức tạp nhất — nhiều inputs, nhiều trạng thái

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** React Router route `/signature` → `SignaturePage.tsx`
- **Từ Prompt 03:** Import `downloadBlob`, `clearSensitiveData`
- **Từ Prompt 05:** Tái sử dụng `FileDropZone`
- **Từ Prompt 07:** Tái sử dụng `PEMKeyDisplay`
- **Từ Prompt 09:** Tái sử dụng `MetadataCard`
- **Từ Prompt 10:** API endpoints: `/api/signature/sign-encrypt`, `/api/signature/decrypt-verify`, `/api/signature/sign-only`, `/api/signature/verify-only`

---

## 🎯 Mục tiêu

Xây dựng trang `/signature` gồm 4 tab:

1. **Tab Tổng quan** (mặc định) — giải thích 3 tính chất + sơ đồ quy trình
2. **Tab Ký & Mã hóa** — nhập 2 key + upload video → download `.signed.enc`
3. **Tab Giải mã & Xác minh** — upload `.signed.enc` + 2 key → kết quả verify
4. **Tab Chỉ ký** — ký video mà không mã hóa (cho so sánh)

---

## 📁 File cần tạo/chỉnh sửa

```
frontend/src/
├── pages/SignaturePage.tsx                         ← Implement đầy đủ
├── components/crypto/
│   ├── SignatureDiagram.tsx                        ← Tạo mới
│   ├── SignEncryptPanel.tsx                        ← Tạo mới
│   ├── DecryptVerifyPanel.tsx                      ← Tạo mới
│   ├── SignOnlyPanel.tsx                           ← Tạo mới
│   └── SignatureVerifyResult.tsx                   ← Tạo mới (REUSABLE)
├── stores/useSignatureStore.ts                     ← Tạo mới
├── schemas/signatureSchema.ts                      ← Tạo mới
└── services/signatureService.ts                    ← Tạo mới
```

---

## ✅ Yêu cầu chức năng

### SignaturePage.tsx

- Tiêu đề: "Chữ ký số (Digital Signature)"
- 3 badges ở đầu trang:
  - 🔒 **Bảo mật** (blue) — Chỉ người nhận đọc được
  - 🛡️ **Toàn vẹn** (green) — Phát hiện file bị sửa đổi
  - ✍️ **Xác thực** (purple) — Xác minh danh tính người gửi
- 4 tab: "📊 Tổng quan", "✍️ Ký & Mã hóa", "🔓 Giải mã & Xác minh", "🔏 Chỉ ký"

### Tab Tổng quan (SignatureDiagram)

**Giải thích 3 tính chất:**

```
┌─────────────┬─────────────┬─────────────┐
│ 🔒 Bảo mật  │ 🛡️ Toàn vẹn │ ✍️ Xác thực │
│             │             │             │
│ Hybrid Enc  │ SHA-256     │ RSA-PSS     │
│ chỉ người   │ hash phát   │ chữ ký      │
│ nhận đọc    │ hiện sửa đổi│ chứng minh  │
│             │             │ danh tính   │
└─────────────┴─────────────┴─────────────┘
```

**Sơ đồ quy trình ký + mã hóa:**

```
NGƯỜI GỬI (Sender):
  [Video] → SHA-256 → [Hash]
  [Hash] → RSA-PSS Sign (Sender Private Key) → [Chữ ký]
  [Video + Chữ ký] → Hybrid Encrypt (Receiver Public Key) → [.signed.enc]

NGƯỜI NHẬN (Receiver):
  [.signed.enc] → Hybrid Decrypt (Receiver Private Key) → [Video + Chữ ký]
  [Video] → SHA-256 → [Hash thực tế]
  [Chữ ký] → RSA-PSS Verify (Sender Public Key) → ✅ / ❌
  [Hash thực tế] vs [Hash kỳ vọng] → ✅ / ❌
```

### Tab Ký & Mã hóa (SignEncryptPanel)

**Layout 2 cột cho keys:**

```
┌── 🔑 Sender (Người gửi) ─┐  ┌── 🔑 Receiver (Người nhận) ─┐
│ Private Key:              │  │ Public Key:                  │
│ [📁 Upload .pem]         │  │ [📁 Upload .pem]             │
│ [📋 Dán key text]        │  │ [📋 Dán key text]            │
│ [🔗 Dùng key từ RSA]     │  │ [🔗 Dùng key từ RSA]        │
│                           │  │                              │
│ ⚠️ Key này dùng để KÝ    │  │ ⚠️ Key này dùng để MÃ HÓA   │
└───────────────────────────┘  └──────────────────────────────┘
```

- Responsive: 2 cột trên desktop, stack trên mobile (< 768px)
- Sau khi cung cấp cả 2 key + upload video:

**Timeline (StatusTimeline):**

```
✅ Tính SHA-256 hash video...              (done, xanh)
🔄 Ký bằng RSA-PSS...                     (active, spinner)
⏸️ Mã hóa AES key bằng RSA                (pending, xám)
⏸️ Mã hóa video bằng AES-GCM              (pending, xám)
```

- Timeline cập nhật theo trạng thái:
  - `pending` (xám + dấu chấm)
  - `active` (xanh dương + spinner)
  - `done` (xanh lá + ✅)
  - `error` (đỏ + ❌)

> **⚠️ Lưu ý quan trọng:** Timeline 4 bước là **visual simulation** (hiển thị tuần tự bằng `setTimeout`). Frontend chỉ gọi **1 API call** duy nhất (`POST /api/signature/sign-encrypt`). Backend KHÔNG gửi progress từng bước. Mục đích: giáo dục người dùng hiểu quy trình nội bộ đang diễn ra.

- Kết quả: download `.signed.enc` + hiện SHA-256 hash

### Tab Giải mã & Xác minh (DecryptVerifyPanel)

- **Upload `.signed.enc`** → parse metadata → MetadataCard
- **Layout 2 cột keys:**
  - Receiver Private Key (để giải mã)
  - Sender Public Key (để xác minh)
- **Kết quả xác minh** → `SignatureVerifyResult` component:

**Trường hợp ✅ HỢP LỆ:**

```
┌─ ✅ Kết quả xác minh ──────────────────────────┐
│  Chữ ký HỢP LỆ                                │
│                                                │
│  SHA-256 Expected: abc123def456...             │
│  SHA-256 Actual:   abc123def456...             │
│  Trạng thái: ✅ Hash khớp                      │
│                                                │
│  📅 Signed at: 2024-01-01 10:30:00             │
│  🔐 Algorithm: RSA-PSS-SHA256                  │
│                                                │
│  [⬇️ Tải video gốc]                           │
└────────────────────────────────────────────────┘
```
Background: `bg-green-50`, border: `border-green-500`

**Trường hợp ❌ KHÔNG HỢP LỆ:**

```
┌─ ❌ Kết quả xác minh ──────────────────────────┐
│  Chữ ký KHÔNG HỢP LỆ                          │
│                                                │
│  ⚠️ CẢNH BÁO: Video có thể đã bị giả mạo!    │
│                                                │
│  SHA-256 Expected: abc123def456...             │
│  SHA-256 Actual:   789xyz...                   │
│  Trạng thái: ❌ Hash KHÔNG khớp                │
│                                                │
│  [⬇️ Tải video (CẢNH BÁO: có thể bị sửa đổi)] │
└────────────────────────────────────────────────┘
```
Background: `bg-red-50`, border: `border-red-500`

### Tab Chỉ ký (SignOnlyPanel)

- Upload video + Private Key
- Nút "🔏 Ký video"
- Kết quả:
  - Hiện SHA-256 hash
  - Hiện signature_b64 (textarea readonly)
  - Nút copy signature
  - Nút download signature file JSON:
    ```json
    {
      "signature_b64": "...",
      "hash_algorithm": "SHA-256",
      "video_hash_hex": "abc123...",
      "algorithm": "RSA-PSS-SHA256"
    }
    ```
  - Trong tab này cũng có phần **Xác minh** (verify-only):
    - Upload video + paste signature + public key
    - Nút "Xác minh" → ✅ hoặc ❌

---

## 🔧 Yêu cầu kỹ thuật

### `stores/useSignatureStore.ts`

```typescript
interface SignatureStore {
  // Sign-Encrypt state
  seFile: File | null
  senderPrivateKey: string
  receiverPublicKey: string
  seProgress: number
  seStatus: 'idle' | 'hashing' | 'signing' | 'encrypting' | 'done' | 'error'
  seBlob: Blob | null
  seVideoHash: string | null

  // Decrypt-Verify state
  dvEncFile: File | null
  dvMetadata: Record<string, any> | null
  receiverPrivateKey: string
  senderPublicKey: string
  dvProgress: number
  dvStatus: 'idle' | 'decrypting' | 'verifying' | 'done' | 'error'
  dvResult: {
    signatureValid: boolean
    expectedHash: string
    actualHash: string
    message: string
  } | null
  dvBlob: Blob | null

  // Sign-Only state
  soFile: File | null
  soPrivateKey: string
  soStatus: 'idle' | 'processing' | 'done' | 'error'
  soResult: {
    signatureB64: string
    videoHashHex: string
  } | null
}
```

### `SignatureVerifyResult.tsx` (REUSABLE)

```typescript
interface SignatureVerifyResultProps {
  valid: boolean
  expectedHash: string
  actualHash: string
  message: string
  algorithm?: string
  signedAt?: string
  onDownload: () => void
}
// Render card xanh (valid) hoặc đỏ (invalid) với SHA-256 hashes
```

### `signatureSchema.ts`

```typescript
export const SenderPrivateKeySchema = z.string()
  .min(1, "Sender private key không được để trống")
  .refine(val =>
    val.includes('-----BEGIN PRIVATE KEY-----') ||
    val.includes('-----BEGIN RSA PRIVATE KEY-----'),
    "Phải là RSA Private Key PEM hợp lệ")

export const ReceiverPublicKeySchema = z.string()
  .min(1, "Receiver public key không được để trống")
  .refine(val => val.includes('-----BEGIN PUBLIC KEY-----'),
    "Phải là RSA Public Key PEM hợp lệ")
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- 3 badges tính chất luôn hiển thị ở đầu trang
- Tab "Tổng quan" là mặc định
- Layout 2 cột cho keys trên desktop, stack trên mobile
- Timeline cập nhật realtime (4 bước)
- SignatureVerifyResult phải phân biệt rõ ✅ và ❌ bằng màu sắc
- Cho phép download video ngay cả khi signature invalid (kèm cảnh báo rõ ràng)
- Hash SHA-256 hiển thị đầy đủ (hex, có nút copy)

❌ **NGHIÊM CẤM:**
- KHÔNG implement crypto logic phía frontend (mọi thứ qua API)
- KHÔNG log private key
- KHÔNG block download khi signature invalid — CHỈ cảnh báo

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý frontend |
|-----------|-------------------|
| Sender key == Receiver key (cùng 1 người test) | Cho phép — không validate |
| Sai receiver private key | API trả 400 → alert "Private key người nhận không đúng" |
| Sai sender public key | API trả valid=false → SignatureVerifyResult đỏ |
| File `.signed.enc` bị truncated | Metadata parse fail → alert lỗi |
| Upload nhầm file `.enc` (không phải `.signed.enc`) | FileDropZone reject |

---

## 🎨 Yêu cầu UI/UX

- **3 badges:** Blue (Bảo mật), Green (Toàn vẹn), Purple (Xác thực)
- **Buttons:** "✍️ Ký & Mã hóa" = blue-600, "🔓 Giải mã" = green-600
- **Verify result:** `bg-green-50 border-green-500` (valid), `bg-red-50 border-red-500` (invalid)
- **Timeline:** Icon spinner (active), ✅ (done), ❌ (error), ⏸️ (pending)
- **Hash display:** Monospace font, full hex, nút copy

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Mở trang | Navigate `/signature` | 3 badges + tab "Tổng quan" active |
| T02 | Sơ đồ | Xem tab Tổng quan | Sơ đồ 2 người + quy trình rõ ràng |
| T03 | 2 cột keys | Tab Ký & Mã hóa trên desktop | Sender bên trái, Receiver bên phải |
| T04 | Mobile layout | Thu nhỏ browser < 768px | 2 cột stack thành 1 |
| T05 | Key từ RSA | Đã tạo key ở `/rsa` | "Dùng key từ RSA" hoạt động |
| T06 | Sign+Encrypt | Upload video + 2 keys | Timeline chạy → download `.signed.enc` |
| T07 | Timeline | Đang xử lý | 4 bước hiện đúng trạng thái |
| T08 | Hash hiển thị | Sau khi ký | SHA-256 hash hiện + nút copy |
| T09 | Decrypt+Verify ✅ | Upload `.signed.enc` + key đúng | Card xanh, hash match |
| T10 | Decrypt+Verify ❌ | Upload + sai sender public | Card đỏ, cảnh báo |
| T11 | Download khi invalid | Khi signature invalid | Nút download vẫn có (kèm cảnh báo) |
| T12 | MetadataCard | Upload file ở tab Giải mã | Card metadata hiện |
| T13 | Sign-Only | Tab Chỉ ký: upload + private key | Hash + signature hiển thị |
| T14 | Copy signature | Click copy signature | Clipboard có base64 |
| T15 | Verify-Only | Trong tab Chỉ ký: xác minh signature | ✅ hoặc ❌ |
| T16 | Tab switching | Chuyển 4 tab | Mượt, state giữ nguyên |

---

## 📋 Checklist tự kiểm tra

- [ ] 3 badges (Bảo mật, Toàn vẹn, Xác thực) hiển thị
- [ ] Tab "Tổng quan" là mặc định
- [ ] Sơ đồ quy trình 2 người (Sender/Receiver) rõ ràng
- [ ] Layout 2 cột keys responsive
- [ ] Timeline 4 bước cập nhật realtime
- [ ] SignatureVerifyResult: xanh (✅) / đỏ (❌) rõ ràng
- [ ] Download video khi invalid (kèm cảnh báo)
- [ ] Hash SHA-256 hiện đầy đủ + copy
- [ ] Sign-Only + Verify-Only hoạt động độc lập
- [ ] Key KHÔNG log, KHÔNG localStorage
- [ ] Tất cả text tiếng Việt
- [ ] TypeScript không lỗi
