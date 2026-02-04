---
sidebar_position: 1
---

# takePhoto

Chụp ảnh hoặc chọn ảnh từ thư viện thiết bị.

## Cú pháp

```javascript
window.WindVane.call(
  'WVCamera',
  'takePhoto',
  {
    mode: 'camera' | 'album' | 'both',
    quality: number,
    maxWidth: number,
    maxHeight: number
  },
  (result) => { /* success */ },
  (error) => { /* fail */ }
);
```

## Tham số đầu vào

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `mode` | `string` | Không | Chế độ: `'camera'` (chỉ chụp), `'album'` (chỉ chọn), `'both'` (cả hai) - Mặc định: `'both'` |
| `quality` | `number` | Không | Chất lượng ảnh (0-100) - Mặc định: 80 |
| `maxWidth` | `number` | Không | Chiều rộng tối đa (px) |
| `maxHeight` | `number` | Không | Chiều cao tối đa (px) |

## Kết quả trả về

**Success callback:**

```javascript
{
  url: string,        // Đường dẫn local của ảnh
  width: number,      // Chiều rộng ảnh
  height: number,     // Chiều cao ảnh
  size: number,       // Kích thước file (bytes)
  type: string        // MIME type (image/jpeg, image/png)
}
```

**Fail callback:**

```javascript
{
  error: string,      // Mã lỗi
  errorMessage: string // Mô tả lỗi
}
```

## Ví dụ

### 1. Chụp ảnh cơ bản

```javascript
window.WindVane.call('WVCamera', 'takePhoto', 
  { mode: 'camera' },
  (result) => {
    console.log('Ảnh đã chụp:', result.url);
    displayImage(result.url);
  },
  (error) => {
    console.error('Lỗi chụp ảnh:', error);
  }
);
```

### 2. Chọn ảnh từ thư viện với nén

```javascript
window.WindVane.call('WVCamera', 'takePhoto', 
  {
    mode: 'album',
    quality: 70,
    maxWidth: 1920,
    maxHeight: 1080
  },
  (result) => {
    console.log('Ảnh đã chọn:', result.url);
    console.log('Kích thước:', result.size, 'bytes');
    uploadImage(result.url);
  },
  (error) => {
    console.error('Lỗi chọn ảnh:', error);
  }
);
```

### 3. Upload avatar với Promise wrapper

```javascript
const takePhoto = (mode = 'both') => {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVCamera', 'takePhoto',
      {
        mode,
        quality: 80,
        maxWidth: 800,
        maxHeight: 800
      },
      resolve,
      reject
    );
  });
};

// Sử dụng
async function updateAvatar() {
  try {
    const photo = await takePhoto('both');
    
    // Hiển thị preview
    document.getElementById('avatar').src = photo.url;
    
    // Upload lên server
    await uploadToServer(photo.url);
    
    showToast('Cập nhật avatar thành công!');
  } catch (error) {
    showToast('Lỗi: ' + error.errorMessage);
  }
}
```

### 4. React Hook

```jsx
import { useState } from 'react';

function useCamera() {
  const [loading, setLoading] = useState(false);
  const [photo, setPhoto] = useState(null);

  const takePhoto = async (mode = 'both') => {
    setLoading(true);
    try {
      const result = await new Promise((resolve, reject) => {
        window.WindVane.call('WVCamera', 'takePhoto',
          { mode, quality: 80, maxWidth: 1920 },
          resolve,
          reject
        );
      });
      setPhoto(result);
      return result;
    } catch (error) {
      console.error('Camera error:', error);
      throw error;
    } finally {
      setLoading(false);
    }
  };

  return { takePhoto, photo, loading };
}

// Component sử dụng
function ProfilePhoto() {
  const { takePhoto, photo, loading } = useCamera();

  return (
    <div>
      {photo && <img src={photo.url} alt="Profile" />}
      <button onClick={() => takePhoto('both')} disabled={loading}>
        {loading ? 'Đang chụp...' : 'Chọn ảnh'}
      </button>
    </div>
  );
}
```

## Use Cases

| Use Case | Cấu hình khuyến nghị |
|----------|----------------------|
| **Avatar/Profile** | `quality: 80, maxWidth: 800, maxHeight: 800` |
| **Photo gallery** | `quality: 85, maxWidth: 1920, maxHeight: 1080` |
| **Document scan** | `quality: 90, mode: 'camera'` |
| **Quick share** | `quality: 70, maxWidth: 1280` |

## Best Practices

### 1. Nén ảnh trước khi upload

```javascript
const compressAndUpload = async (mode) => {
  const photo = await takePhoto(mode);
  
  // Nếu ảnh quá lớn, nén thêm
  if (photo.size > 2 * 1024 * 1024) { // > 2MB
    console.warn('Ảnh lớn, nên nén thêm');
  }
  
  await uploadToServer(photo.url);
};
```

### 2. Validation

```javascript
const validatePhoto = (photo) => {
  // Kiểm tra kích thước
  if (photo.size > 5 * 1024 * 1024) {
    throw new Error('Ảnh không được vượt quá 5MB');
  }
  
  // Kiểm tra định dạng
  if (!['image/jpeg', 'image/png'].includes(photo.type)) {
    throw new Error('Chỉ chấp nhận ảnh JPG hoặc PNG');
  }
  
  // Kiểm tra tỷ lệ
  const ratio = photo.width / photo.height;
  if (ratio < 0.5 || ratio > 2) {
    console.warn('Tỷ lệ ảnh không chuẩn');
  }
  
  return true;
};
```

### 3. Error handling

```javascript
const handleCameraError = (error) => {
  switch (error.error) {
    case 'PERMISSION_DENIED':
      showToast('Vui lòng cấp quyền truy cập camera');
      break;
    case 'USER_CANCELLED':
      // Không cần thông báo, user tự hủy
      break;
    case 'NO_CAMERA':
      showToast('Thiết bị không có camera');
      break;
    default:
      showToast('Lỗi không xác định: ' + error.errorMessage);
  }
};
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Xử lý |
|--------|-------------|-------|
| `PERMISSION_DENIED` | Chưa cấp quyền camera | Yêu cầu user vào Settings |
| `USER_CANCELLED` | User hủy chụp/chọn ảnh | Không cần xử lý |
| `NO_CAMERA` | Thiết bị không có camera | Chỉ cho phép chọn từ album |
| `FILE_TOO_LARGE` | Ảnh quá lớn | Nén với quality thấp hơn |

## Quyền yêu cầu

- **Camera**: Cần approve trong Dashboard
- **Photo Library**: Cần approve trong Dashboard
- **User consent**: Hiển thị dialog xin quyền lần đầu

:::tip Khuyến nghị
- Luôn nén ảnh trước khi upload để tiết kiệm băng thông
- Sử dụng `maxWidth` và `maxHeight` để giới hạn kích thước
- Quality 70-80 là cân bằng tốt giữa chất lượng và dung lượng
- Hiển thị preview trước khi upload
:::

## Xem thêm

- [chooseVideo](./chooseVideo) - Chọn video
- [File Upload](../D_file_storage/uploadFile) - Upload file lên server
