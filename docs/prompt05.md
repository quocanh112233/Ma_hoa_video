# Prompt 05 – AES Frontend: React UI Mã hóa/Giải mã AES

---

## 📝 TL;DR

- **Input:** Backend AES API từ Prompt 04 đang hoạt động
- **Output:** Trang `/aes` hoàn chỉnh với 2 tab Mã hóa/Giải mã
- **Files to create:** `AESPage.tsx`, `AESEncryptPanel.tsx`, `AESDecryptPanel.tsx`, `FileDropZone.tsx`, `useAESStore.ts`, `aesSchema.ts`, `aesService.ts`
- **Key constraint:** Zustand cho state, Zod cho validation, Axios cho API calls
- **Definition of Done:** Encrypt/Decrypt round-trip hoạt động end-to-end + 15 test cases pass

---

## 🎭 Role

Bạn là **Frontend Engineer** chuyên về React/TypeScript. Nhiệm vụ là xây dựng giao diện trang **Mã hóa AES-256-GCM**, bao gồm chọn file video, quản lý key, hiển thị progress bar, và download kết quả — tất cả tích hợp với API backend từ Prompt 04.

---

## 📋 Context

- Phase 1 Frontend — Prompt 04 (AES Backend) đã hoàn thành
- AES-256-GCM là thuật toán đối xứng: cùng 1 key để encrypt và decrypt
- Người dùng CÓ THỂ tự tạo key mới hoặc nhập key có sẵn
- Kết quả mã hóa: 2 file — `.enc` (video encrypted) và `.key` (key JSON)
- Route: `/aes`

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** React Router route `/aes` → `AESPage.tsx`
- **Từ Prompt 03:** Import `downloadBlob`, `clearSensitiveData` từ `utils/security.ts`
- **Từ Prompt 04:** API endpoints: `GET /api/aes/generate-key`, `POST /api/aes/encrypt`, `POST /api/aes/decrypt`
- **KHÔNG thay đổi:** Không sửa bất kỳ file backend nào

---

## 🎯 Mục tiêu

Xây dựng trang `/aes` gồm 2 tab:

1. **Tab Mã hóa** — upload video + chọn key → download `.enc` + `.key`
2. **Tab Giải mã** — upload `.enc` + nhập key → download video gốc

---

## 📁 File cần tạo/chỉnh sửa

```
frontend/src/
├── pages/AESPage.tsx                        ← Implement đầy đủ
├── components/crypto/
│   ├── AESEncryptPanel.tsx                  ← Tạo mới
│   └── AESDecryptPanel.tsx                  ← Tạo mới
├── components/shared/
│   └── FileDropZone.tsx                     ← Tạo mới (REUSABLE)
├── stores/useAESStore.ts                    ← Tạo mới
├── schemas/aesSchema.ts                     ← Tạo mới
└── services/aesService.ts                   ← Tạo mới
```

---

## ✅ Yêu cầu chức năng

### AESPage.tsx

- Tiêu đề: "Mã hóa AES-256-GCM"
- 2 tabs: "🔒 Mã hóa" (active mặc định) | "🔓 Giải mã"
- Mỗi tab render panel component tương ứng

### Tab Mã hóa (AESEncryptPanel)

**Bước 1 – Chọn video:**
- `FileDropZone` component (reusable): hỗ trợ drag & drop + click chọn file
- Chỉ nhận: `.mp4`, `.mkv`, `.avi`, `.mov`, `.webm`
- Hiển thị: tên file, kích thước (human-readable: MB/GB)
- Nếu file sai định dạng → hiện lỗi ngay (Zod validation)

**Bước 2 – Quản lý key:**
- Radio: `◉ Tạo key tự động` (mặc định) | `○ Nhập key thủ công`
- Nếu "Tạo tự động": Nút "🔑 Tạo key mới" → gọi `/api/aes/generate-key` → hiện key trong input readonly
- Nếu "Nhập thủ công": Input textarea để paste key base64
- Copy button (📋) bên cạnh key input

**Bước 3 – Mã hóa:**
- Nút "🔒 Bắt đầu mã hóa" (disabled nếu chưa chọn file hoặc chưa có key)
- Progress bar cập nhật realtime (`onUploadProgress` của Axios)
- Trạng thái text: "Đang tải lên..." → "Đang mã hóa..." → "Hoàn thành!"

**Kết quả (sau khi xong):**
- Nút "⬇️ Tải file .enc"
- Nút "⬇️ Tải file .key"
- Cảnh báo vàng (amber): `"⚠️ Hãy lưu file .key cẩn thận! Mất key = Mất video."`
- Nút "Mã hóa video khác" → reset toàn bộ form

### Tab Giải mã (AESDecryptPanel)

**Bước 1 – Upload file `.enc`:**
- `FileDropZone` nhận `.enc`
- Hiển thị tên file và kích thước

**Bước 2 – Nhập key:**
- Radio: `◉ Upload file .key` | `○ Nhập key base64 thủ công`
- Nếu upload `.key`: parse JSON và lấy `key_b64`
- Nếu nhập thủ công: textarea

**Bước 3 – Giải mã:**
- Nút "🔓 Giải mã"
- Progress bar
- Sau khi xong: nút download video gốc
- Nếu sai key → alert đỏ: `"❌ Giải mã thất bại: Key sai hoặc file bị hỏng"`

---

## 🔧 Yêu cầu kỹ thuật

### `stores/useAESStore.ts` (Zustand)

```typescript
interface AESStore {
  // Tab state
  activeTab: 'encrypt' | 'decrypt'

  // Encrypt state
  encryptFile: File | null
  keyMode: 'auto' | 'manual'
  keyB64: string
  encryptProgress: number
  encryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  encryptedBlob: Blob | null
  encryptedFilename: string | null
  encryptError: string | null

  // Decrypt state
  decryptEncFile: File | null
  decryptKeyB64: string
  decryptKeyMode: 'file' | 'manual'
  decryptProgress: number
  decryptStatus: 'idle' | 'uploading' | 'processing' | 'done' | 'error'
  decryptedBlob: Blob | null
  decryptedFilename: string | null
  decryptError: string | null

  // Actions
  setActiveTab: (tab: 'encrypt' | 'decrypt') => void
  setEncryptFile: (file: File | null) => void
  setKeyMode: (mode: 'auto' | 'manual') => void
  setKeyB64: (key: string) => void
  resetEncrypt: () => void
  resetDecrypt: () => void
}
```

### `schemas/aesSchema.ts` (Zod)

```typescript
import { z } from 'zod';

export const VideoFileSchema = z.instanceof(File)
  .refine(f => f.size > 0, "File không được rỗng")
  .refine(
    f => ['mp4', 'mkv', 'avi', 'mov', 'webm'].some(ext => f.name.toLowerCase().endsWith(`.${ext}`)),
    "Chỉ chấp nhận file video (.mp4, .mkv, .avi, .mov, .webm)"
  )

export const AESKeySchema = z.string()
  .min(1, "Key không được để trống")
  .refine(val => {
    try {
      const decoded = atob(val.trim());
      return decoded.length === 32;
    } catch { return false; }
  }, "Key phải là base64 hợp lệ (32 bytes / 256 bit)")

export const EncFileSchema = z.instanceof(File)
  .refine(f => f.name.endsWith('.enc'), "Phải là file .enc")
```

### `services/aesService.ts`

```typescript
import api from './api';

export async function generateKey(): Promise<{ key_b64: string }> {
  const { data } = await api.get('/api/aes/generate-key');
  return data;
}

export async function encryptVideo(
  file: File,
  keyB64?: string,
  onProgress?: (percent: number) => void
): Promise<{ blob: Blob; keyB64: string; filename: string }> {
  const formData = new FormData();
  formData.append('file', file);
  if (keyB64) formData.append('key_b64', keyB64);

  const response = await api.post('/api/aes/encrypt', formData, {
    responseType: 'blob',
    onUploadProgress: (e) => {
      if (e.total && onProgress) onProgress(Math.round((e.loaded / e.total) * 100));
    }
  });

  const responseKeyB64 = response.headers['x-key-b64'] || keyB64 || '';
  const filename = response.headers['content-disposition']?.match(/filename="(.+)"/)?.[1] || 'video.enc';

  return { blob: response.data, keyB64: responseKeyB64, filename };
}

export async function decryptVideo(
  encFile: File,
  keyB64: string,
  onProgress?: (percent: number) => void
): Promise<{ blob: Blob; filename: string }> {
  const formData = new FormData();
  formData.append('enc_file', encFile);
  formData.append('key_b64', keyB64);

  const response = await api.post('/api/aes/decrypt', formData, {
    responseType: 'blob',
    onUploadProgress: (e) => {
      if (e.total && onProgress) onProgress(Math.round((e.loaded / e.total) * 100));
    }
  });

  const filename = response.headers['content-disposition']?.match(/filename="(.+)"/)?.[1] || 'video.mp4';
  return { blob: response.data, filename };
}
```

### `components/shared/FileDropZone.tsx` (REUSABLE)

- Props:
  ```typescript
  interface FileDropZoneProps {
    accept: string[]        // ['.mp4', '.mkv', ...] hoặc ['.enc']
    maxSizeMB?: number     // Optional: giới hạn size
    onFileSelect: (file: File) => void
    label?: string
    description?: string
  }
  ```
- States visual:
  - **idle:** border dashed gray, icon upload
  - **dragover:** border solid blue, background highlight
  - **selected:** hiển thị tên file + kích thước + nút ❌ xóa
  - **error:** border red, message lỗi

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- `FileDropZone` phải reusable (sẽ dùng lại ở trang RSA, Hybrid, Signature)
- Zod validation chạy TRƯỚC khi gọi API (client-side validation)
- Progress bar phải dùng `onUploadProgress` callback
- Nút `disabled` khi `status !== 'idle'` để tránh double-submit
- `downloadBlob()` từ `security.ts` để download an toàn
- Sau khi download: `clearSensitiveData()` để xóa key khỏi state

❌ **NGHIÊM CẤM:**
- KHÔNG dùng `console.log(key)` hay log bất kỳ key nào
- KHÔNG lưu key vào `localStorage` hoặc `sessionStorage`
- KHÔNG implement logic crypto phía frontend — mọi thứ qua API
- KHÔNG dùng `any` type trong TypeScript (trừ khi bắt buộc cho third-party lib)

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý frontend |
|-----------|-------------------|
| Backend trả 400 (sai key) | Hiện alert đỏ message từ `response.data.detail` |
| Backend trả 413 (file quá lớn) | Hiện alert đỏ "File quá lớn" |
| Backend trả 429 (rate limit) | Hiện alert "Vui lòng đợi 1 phút" |
| Network error (mất kết nối) | Hiện alert "Không thể kết nối server" |
| File upload timeout (video quá lớn) | Axios timeout 5 phút, hiện lỗi cụ thể |
| User chọn sai loại file | Zod validation lỗi ngay, không gọi API |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG tạo `FileDropZone` inline trong Panel — tách thành component riêng
2. KHÔNG gọi API trong `useEffect` khi component mount — chỉ gọi khi user click
3. KHÔNG dùng `window.prompt()` để nhập key — dùng controlled input
4. KHÔNG `setState` trong loop — batch state updates
5. KHÔNG dùng `fetch()` — dùng Axios (đã config baseURL + timeout)

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Mở trang | Navigate to `/aes` | 2 tabs hiển thị, tab "Mã hóa" active |
| T02 | Drag & drop | Kéo file `.mp4` vào FileDropZone | File được chọn, hiện tên + kích thước |
| T03 | Sai format | Chọn file `.txt` | Zod lỗi ngay, không gọi API |
| T04 | Tạo key | Click "Tạo key mới" | Key base64 hiện trong input readonly |
| T05 | Encrypt auto | Chọn file + auto key → encrypt | Progress bar, download link .enc + .key |
| T06 | Download .enc | Click download | File `.enc` tải về thành công |
| T07 | Download .key | Click download | File `.key` (JSON) tải về |
| T08 | Cảnh báo key | Sau khi mã hóa xong | Thấy box cảnh báo vàng |
| T09 | Decrypt đúng | Tab Giải mã: upload .enc + .key đúng | Download video gốc, xem được |
| T10 | Decrypt sai key | Upload .enc + key sai | Alert đỏ "Giải mã thất bại" |
| T11 | Key thủ công | Nhập key base64 hợp lệ → encrypt | Thành công |
| T12 | Key sai format | Nhập "hello" vào key field → encrypt | Zod lỗi, không gọi API |
| T13 | File lớn | Upload video >500MB | Progress bar cập nhật, không crash |
| T14 | Reset form | Click "Mã hóa video khác" | Form reset hoàn toàn |
| T15 | Tab switching | Click tab Mã hóa ↔ Giải mã | Chuyển mượt, state giữ nguyên |

---

## 🎨 Yêu cầu UI/UX

- Màu sắc: button mã hóa `blue-600`, button giải mã `green-600`
- Cảnh báo: amber (key warning), red (lỗi)
- Progress bar: gradient blue, có % text
- FileDropZone: border dashed, hover effect, drag highlight
- Toàn bộ text tiếng Việt

---

## 📋 Checklist tự kiểm tra

- [ ] 2 tabs hoạt động, chuyển mượt
- [ ] FileDropZone reusable, hỗ trợ drag & drop + click
- [ ] Zod validation chạy trước khi gọi API
- [ ] Progress bar cập nhật realtime
- [ ] Download 2 file: .enc và .key (khi auto key)
- [ ] Cảnh báo vàng về lưu key
- [ ] Giải mã đúng key → video xem được
- [ ] Giải mã sai key → alert đỏ
- [ ] Nút disabled khi đang xử lý
- [ ] Key KHÔNG xuất hiện trong console.log
- [ ] Key KHÔNG lưu vào localStorage
- [ ] Toàn bộ text tiếng Việt
- [ ] TypeScript không có lỗi (`npm run build`)
