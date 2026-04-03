# Prompt 07 – RSA Frontend: React UI Mã hóa/Giải mã RSA

---

## 📝 TL;DR

- **Input:** Backend RSA API từ Prompt 06 đang hoạt động
- **Output:** Trang `/rsa` hoàn chỉnh với 3 tab: Sinh Key / Mã hóa / Giải mã
- **Files to create:** `RSAPage.tsx`, `RSAKeyGenPanel.tsx`, `RSAEncryptPanel.tsx`, `RSADecryptPanel.tsx`, `PEMKeyDisplay.tsx`, `useRSAStore.ts`, `rsaSchema.ts`, `rsaService.ts`
- **Key constraint:** Banner cảnh báo học thuật, private key blur, timer mã hóa, giới hạn 5MB
- **Definition of Done:** 15 test cases pass + Security Checklist S06,S08,S10,S12-S14 pass

---

## 🎭 Role

Bạn là **Frontend Engineer** chuyên về React. Nhiệm vụ là xây dựng giao diện trang **Mã hóa RSA**, bao gồm sinh cặp key RSA, mã hóa file nhỏ bằng public key, giải mã bằng private key, và hiển thị rõ giới hạn học thuật của phương pháp này.

---

## 📋 Context

- Phase 2 Frontend — Prompt 06 (RSA Backend) đã hoàn thành
- RSA chỉ hỗ trợ file ≤ 5MB (học thuật) — người dùng cần được thông báo rõ
- Key được quản lý dưới dạng file PEM hoặc text
- `useRSAStore` sẽ được **tái sử dụng** ở trang Hybrid (Prompt 09) để share public key
- Route: `/rsa`

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** React Router route `/rsa` → `RSAPage.tsx`
- **Từ Prompt 03:** Import `downloadBlob`, `clearSensitiveData` từ `utils/security.ts`
- **Từ Prompt 05:** Tái sử dụng `FileDropZone` component từ `components/shared/`
- **Từ Prompt 06:** API endpoints: `/api/rsa/generate-keypair`, `/api/rsa/encrypt`, `/api/rsa/decrypt`, `/api/rsa/validate-keypair`
- **KHÔNG thay đổi:** Không sửa bất kỳ file AES nào

---

## 🎯 Mục tiêu

Xây dựng trang `/rsa` gồm 3 tab:

1. **Tab Sinh Key** – tạo cặp RSA key mới, download PEM
2. **Tab Mã hóa** – upload video ≤5MB + public key → download `.rsa.enc`
3. **Tab Giải mã** – upload `.rsa.enc` + private key → download file gốc

---

## 📁 File cần tạo/chỉnh sửa

```
frontend/src/
├── pages/RSAPage.tsx                        ← Implement đầy đủ
├── components/crypto/
│   ├── RSAKeyGenPanel.tsx                   ← Tạo mới
│   ├── RSAEncryptPanel.tsx                  ← Tạo mới
│   ├── RSADecryptPanel.tsx                  ← Tạo mới
│   └── PEMKeyDisplay.tsx                    ← Tạo mới (REUSABLE)
├── stores/useRSAStore.ts                    ← Tạo mới
├── schemas/rsaSchema.ts                     ← Tạo mới
└── services/rsaService.ts                   ← Tạo mới
```

---

## ✅ Yêu cầu chức năng

### RSAPage.tsx

- Tiêu đề: "Mã hóa RSA-2048"
- Banner cảnh báo học thuật màu **amber** ở đầu trang (KHÔNG thể đóng):

  ```
  ⚠️ Lưu ý học thuật: RSA chỉ mã hóa được dữ liệu rất nhỏ (~190 bytes mỗi lần).
  Mã hóa video trực tiếp bằng RSA cực kỳ chậm và không thực tế.
  Phase này chỉ dùng cho mục đích học tập và so sánh.
  Giới hạn file: 5MB
  ```

- 3 tab: "🔑 Sinh Key", "🔒 Mã hóa", "🔓 Giải mã"

### Tab Sinh Key (RSAKeyGenPanel)

- Select key size: RSA-2048 (mặc định) / RSA-4096
- Nút "Tạo cặp Key RSA"
- Loading spinner khi đang tạo (4096 có thể mất vài giây)
- Sau khi tạo, hiển thị:
  - **Public Key:** `PEMKeyDisplay` readonly, nút "📋 Copy" + "⬇️ Tải Public Key (.pem)"
  - **Private Key:** `PEMKeyDisplay` + blur/ẩn mặc định, nút "👁️ Hiện/Ẩn", "📋 Copy", "⬇️ Tải Private Key (.pem)"
  - Cảnh báo đỏ: "🔴 KHÔNG chia sẻ Private Key với bất kỳ ai!"
- Nút "Kiểm tra cặp key" (gọi `/api/rsa/validate-keypair`): hiển thị ✅ hoặc ❌

### Tab Mã hóa (RSAEncryptPanel)

- FileDropZone: nhận `.mp4`, `.mkv`, `.avi`, `.mov`, `.webm` **≤ 5MB**
  - Nếu file > 5MB: cảnh báo ngay "File vượt quá 5MB. RSA chỉ hỗ trợ file ≤ 5MB."
- Cung cấp Public Key:
  - Radio: "📁 Upload file .pem" HOẶC "📋 Dán key text"
  - Nếu dán: textarea để nhập PEM
- Nút "🔒 Mã hóa bằng RSA"
- Progress bar upload
- **Timer:** hiển thị thời gian mã hóa đếm giây realtime (setInterval 100ms)
- Sau khi xong: nút download `.rsa.enc` + thông tin chunks đã xử lý
- **Hint so sánh:** "💡 So sánh: AES mã hóa cùng file này chỉ mất <1 giây"

### Tab Giải mã (RSADecryptPanel)

- FileDropZone: nhận `.rsa.enc`
- Cung cấp Private Key:
  - Radio: "📁 Upload file .pem" HOẶC "📋 Dán key text"
  - Nếu nhập text: textarea + toggle hiện/ẩn
- Nút "🔓 Giải mã bằng RSA"
- Progress bar
- Sau khi xong: nút download file gốc

---

## 🔧 Yêu cầu kỹ thuật

### `stores/useRSAStore.ts` (Zustand)

```typescript
interface RSAStore {
  // KeyGen state
  keySize: 2048 | 4096
  publicKeyPem: string | null
  privateKeyPem: string | null
  showPrivateKey: boolean
  keyGenStatus: 'idle' | 'generating' | 'done' | 'error'

  // Encrypt state
  encryptFile: File | null
  publicKeyInput: string
  encryptProgress: number
  encryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  encryptedBlob: Blob | null
  encryptError: string | null
  encryptDuration: number  // giây (float)

  // Decrypt state
  decryptEncFile: File | null
  privateKeyInput: string
  decryptProgress: number
  decryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  decryptedBlob: Blob | null
  decryptedFilename: string | null
  decryptError: string | null

  // Actions
  setKeySize: (size: 2048 | 4096) => void
  setShowPrivateKey: (show: boolean) => void
  resetKeyGen: () => void
  resetEncrypt: () => void
  resetDecrypt: () => void
}
```

> ⚠️ **QUAN TRỌNG:** `publicKeyPem` và `privateKeyPem` sẽ được đọc bởi `useHybridStore` ở Prompt 09 (nút "Dùng key từ trang RSA"). Đảm bảo chúng là top-level state, dễ truy cập.

### `schemas/rsaSchema.ts` (Zod)

```typescript
export const PEMPublicKeySchema = z.string()
  .min(1, "Public key không được để trống")
  .refine(val => val.includes('-----BEGIN PUBLIC KEY-----'),
    "Phải là PEM Public Key hợp lệ")

export const PEMPrivateKeySchema = z.string()
  .min(1, "Private key không được để trống")
  .refine(val =>
    val.includes('-----BEGIN PRIVATE KEY-----') ||
    val.includes('-----BEGIN RSA PRIVATE KEY-----'),
    "Phải là PEM Private Key hợp lệ")

export const VideoFileSmallSchema = z.instanceof(File)
  .refine(f => f.size <= 5 * 1024 * 1024, "File phải ≤ 5MB (giới hạn RSA)")
  .refine(f => ['mp4','mkv','avi','mov','webm'].some(ext => f.name.toLowerCase().endsWith(`.${ext}`)),
    "Chỉ chấp nhận file video")
```

### `components/crypto/PEMKeyDisplay.tsx` (REUSABLE)

- Props: `label: string`, `pem: string`, `isPrivate?: boolean`
- Textarea readonly
- Nút Copy (copy to clipboard)
- Nút Download (download as .pem)
- Nếu `isPrivate`: blur mặc định + toggle hiện/ẩn (`filter: blur(6px)`)

### Timer khi mã hóa

```typescript
// Start timer khi bắt đầu mã hóa
const startTime = Date.now();
const timer = setInterval(() => {
  setDuration((Date.now() - startTime) / 1000);
}, 100);

// Stop timer khi xong
clearInterval(timer);
// Hiển thị: `Đã mã hóa trong ${duration.toFixed(1)} giây`
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Banner cảnh báo học thuật LUÔN hiển thị (không có nút đóng)
- Private key PHẢI blur mặc định (`filter: blur(6px)` hoặc `type="password"`)
- Timer đếm giây realtime khi mã hóa
- File > 5MB PHẢI bị block ngay ở frontend (Zod), không gọi API
- PEM validation trước khi gọi API (Zod)

❌ **NGHIÊM CẤM:**
- KHÔNG cho phép file > 5MB upload
- KHÔNG hiển thị private key plaintext mặc định
- KHÔNG log private key ra console

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý frontend |
|-----------|-------------------|
| RSA-4096 generate chậm | Spinner loading + disable button |
| File 5.1MB chọn | Zod reject ngay "File phải ≤ 5MB" |
| PEM text có extra newlines | `.trim()` trước validate |
| API rate limited | Hiện thông báo "Vui lòng đợi" |
| Validate keypair sai | Hiện ❌ badge + message |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG cho phép user bỏ qua banner cảnh báo bằng nút (X)
2. KHÔNG để private key hiện mặc định — PHẢI blur
3. KHÔNG dùng `setTimeout` cho timer — dùng `setInterval` 100ms cho smooth
4. KHÔNG duplicate FileDropZone — import từ `shared/`

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Mở trang | Navigate `/rsa` | Banner cảnh báo amber hiện ngay |
| T02 | Sinh RSA-2048 | Tab Sinh Key → click "Tạo cặp Key RSA" | Public + private key PEM hiện |
| T03 | Private key ẩn | Xem sau T02 | Private key bị blur |
| T04 | Toggle private key | Click "👁️ Hiện" | Private key rõ |
| T05 | Copy public key | Click 📋 trên public key | Clipboard có PEM |
| T06 | Download PEM | Click download public/private | File `.pem` tải về |
| T07 | Validate đúng | Click "Kiểm tra cặp key" | ✅ "Cặp key hợp lệ" |
| T08 | Validate sai | Mix 2 cặp key khác nhau | ❌ "Cặp key không khớp" |
| T09 | File > 5MB | Upload video 10MB | Cảnh báo ngay "File vượt quá 5MB" |
| T10 | Mã hóa | Upload ~1MB + public key | Timer chạy, download link |
| T11 | Timer | Đang mã hóa file 5MB | Đồng hồ đếm giây |
| T12 | Hint so sánh | Sau khi mã hóa xong | "AES mã hóa chỉ mất <1s" |
| T13 | Giải mã đúng | Upload `.rsa.enc` + private key | Video gốc download |
| T14 | Giải mã sai key | Upload + sai private key | Alert đỏ lỗi |
| T15 | PEM sai | Nhập text bừa → mã hóa | Zod validation lỗi |

---

## 📋 Checklist tự kiểm tra

- [ ] Banner cảnh báo học thuật luôn hiển thị, không thể đóng
- [ ] Private key bị blur mặc định
- [ ] Timer đếm giây realtime khi mã hóa
- [ ] File > 5MB bị block ngay (Zod)
- [ ] PEM validation (Zod) hoạt động
- [ ] Download 2 file PEM riêng biệt
- [ ] Validate keypair gọi API, hiện ✅ hoặc ❌
- [ ] Giải mã đúng key → download video gốc
- [ ] Hint so sánh tốc độ vs AES hiện sau mã hóa
- [ ] Key KHÔNG log ra console
- [ ] Key KHÔNG lưu localStorage
- [ ] Toàn bộ text tiếng Việt
- [ ] TypeScript không lỗi
