---
sidebar_position: 1
---

# takePhoto

Chụp ảnh hoặc chọn ảnh từ thư viện thiết bị.

## Cú pháp

```javascript
window.WindVane.call('WVCamera', 'takePhoto', params, successCallback, failCallback);
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `mode` | `string` | Không | `'camera'`: Chỉ chụp ảnh<br/>`'album'`: Chỉ chọn từ thư viện<br/>`'both'`: Cho phép cả hai (mặc định) |
| `quality` | `number` | Không | Chất lượng ảnh (0-100), mặc định 80 |
| `maxWidth` | `number` | Không | Chiều rộng tối đa (px) |
| `maxHeight` | `number` | Không | Chiều cao tối đa (px) |

## Success Callback

```javascript
{
  url: string,        // Đường dẫn local của ảnh
  width: number,      // Chiều rộng ảnh (px)
  height: number,     // Chiều cao ảnh (px)
  size: number        // Kích thước file (bytes)
}
```

## Ví dụ

### Chụp ảnh mới

```javascript
window.WindVane.call('WVCamera', 'takePhoto', 
  { mode: 'camera', quality: 80 },
  (result) => {
    console.log('Ảnh đã chụp:', result.url);
    // Hiển thị preview
    document.getElementById('preview').src = result.url;
  },
  (error) => {
    console.error('Lỗi:', error);
  }
);
```

### Chọn từ thư viện với giới hạn kích thước

```javascript
window.WindVane.call('WVCamera', 'takePhoto', 
  { 
    mode: 'album',
    maxWidth: 1024,
    maxHeight: 1024,
    quality: 70
  },
  (result) => {
    if (result.size > 5 * 1024 * 1024) { // > 5MB
      alert('Ảnh quá lớn, vui lòng chọn ảnh khác');
      return;
    }
    uploadImage(result.url);
  },
  (error) => {
    console.error('Lỗi:', error);
  }
);
```

### Upload ảnh lên server

```javascript
async function uploadAvatar() {
  try {
    // Bước 1: Lấy ảnh
    const result = await new Promise((resolve, reject) => {
      window.WindVane.call('WVCamera', 'takePhoto', 
        { mode: 'both', quality: 70, maxWidth: 800 },
        resolve,
        reject
      );
    });

    // Bước 2: Upload
    const formData = new FormData();
    const response = await fetch(result.url);
    const blob = await response.blob();
    formData.append('avatar', blob, 'avatar.jpg');

    const uploadResponse = await fetch('/api/upload-avatar', {
      method: 'POST',
      body: formData
    });

    const data = await uploadResponse.json();
    console.log('Upload thành công:', data.imageUrl);
  } catch (error) {
    console.error('Lỗi:', error);
  }
}
```

## Best Practices

### 1. Nén ảnh trước khi upload

```javascript
const compressImage = (imageUrl, quality = 70) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      
      // Giữ tỷ lệ, max 1024px
      let width = img.width;
      let height = img.height;
      const maxSize = 1024;
      
      if (width > height && width > maxSize) {
        height = (height * maxSize) / width;
        width = maxSize;
      } else if (height > maxSize) {
        width = (width * maxSize) / height;
        height = maxSize;
      }
      
      canvas.width = width;
      canvas.height = height;
      ctx.drawImage(img, 0, 0, width, height);
      
      canvas.toBlob((blob) => resolve(blob), 'image/jpeg', quality / 100);
    };
    img.src = imageUrl;
  });
};
```

### 2. Kiểm tra kích thước trước khi xử lý

```javascript
function validateImageSize(result) {
  const MAX_SIZE = 10 * 1024 * 1024; // 10MB
  
  if (result.size > MAX_SIZE) {
    alert(`Ảnh quá lớn (${(result.size / 1024 / 1024).toFixed(2)}MB). Tối đa 10MB.`);
    return false;
  }
  return true;
}
```

### 3. Hiển thị preview với loading state

```javascript
function showImagePreview(imageUrl) {
  const preview = document.getElementById('preview');
  const loading = document.getElementById('loading');
  
  loading.style.display = 'block';
  preview.style.display = 'none';
  
  const img = new Image();
  img.onload = () => {
    preview.src = imageUrl;
    preview.style.display = 'block';
    loading.style.display = 'none';
  };
  img.src = imageUrl;
}
```

## Xử lý lỗi

```javascript
window.WindVane.call('WVCamera', 'takePhoto', { mode: 'both' },
  (result) => { /* success */ },
  (error) => {
    if (error.error === 'USER_CANCEL') {
      console.log('Người dùng hủy');
      return;
    }
    
    if (error.error === 'NO_PERMISSION') {
      alert('Vui lòng cấp quyền Camera trong Settings');
      return;
    }
    
    console.error('Lỗi không xác định:', error);
  }
);
```

## Giới hạn

- Ảnh được lưu tạm trong cache của miniapp
- File tạm sẽ bị xóa khi miniapp đóng
- Không thể truy cập thư mục Photos/Albums trực tiếp
- Trên iOS, cần quyền Camera và Photo Library

## Liên quan

- [chooseVideo](./chooseVideo) - Chọn video từ thiết bị
- [File: uploadFile](../D_file_storage/uploadFile) - Upload file lên server
