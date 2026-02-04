---
sidebar_position: 0
id: index
---

# File & Storage APIs

Các API để quản lý file và lưu trữ dữ liệu.

## Danh sách APIs

### File Operations
- **chooseFiles** - Mở trình chọn file
- **read** / **write** - Đọc/ghi file local
- **downloadFile** - Tải file từ URL
- **uploadFile** - Upload file lên server

### Storage
- **setStorage** / **getStorage** - Lưu/lấy dữ liệu local
- **removeStorage** - Xóa dữ liệu
- **clearStorage** - Xóa toàn bộ

## Use Cases phổ biến

### 1. Upload ảnh sản phẩm

```javascript
const uploadProductImage = async (productId) => {
  // Chọn ảnh
  window.WindVane.call('WVFile', 'chooseFiles', 
    { accept: 'image/*', count: 1 },
    async (result) => {
      const filePath = result.files[0].path;
      
      // Upload
      window.WindVane.call('WVFile', 'uploadFile', {
        url: `https://api.example.com/products/${productId}/images`,
        filePath,
        name: 'image'
      }, (uploadResult) => {
        showToast('Đã upload ảnh sản phẩm');
      });
    }
  );
};
```

### 2. Download và lưu file PDF

```javascript
const downloadPDF = (url, filename) => {
  window.WindVane.call('WVFile', 'downloadFile', 
    { url, fileName: filename },
    (result) => {
      showToast('Đã tải file về: ' + result.filePath);
    }
  );
};
```

### 3. Lưu trữ cấu hình local

```javascript
const saveUserPreferences = (prefs) => {
  window.WindVane.call('WVStorage', 'setStorage', {
    key: 'userPreferences',
    data: JSON.stringify(prefs)
  }, () => {
    console.log('Preferences saved');
  });
};

const loadUserPreferences = () => {
  window.WindVane.call('WVStorage', 'getStorage', 
    { key: 'userPreferences' },
    (result) => {
      const prefs = JSON.parse(result.data);
      applyPreferences(prefs);
    }
  );
};
```

### 4. Cache dữ liệu

```javascript
const CacheManager = {
  set(key, value, ttl = 3600000) { // TTL mặc định 1 giờ
    const item = {
      value,
      expiry: Date.now() + ttl
    };
    
    window.WindVane.call('WVStorage', 'setStorage', {
      key,
      data: JSON.stringify(item)
    });
  },

  get(key) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVStorage', 'getStorage', 
        { key },
        (result) => {
          const item = JSON.parse(result.data);
          
          // Kiểm tra expiry
          if (Date.now() > item.expiry) {
            this.remove(key);
            reject(new Error('Cache expired'));
          } else {
            resolve(item.value);
          }
        },
        reject
      );
    });
  },

  remove(key) {
    window.WindVane.call('WVStorage', 'removeStorage', { key });
  }
};

// Sử dụng
CacheManager.set('products', productList, 30 * 60 * 1000); // Cache 30 phút
const products = await CacheManager.get('products');
```

## Giới hạn

| Loại | Giới hạn |
|------|----------|
| **Storage size** | ~10MB mỗi miniapp |
| **Upload size** | Tùy thuộc server (thường ~50MB) |
| **File types** | Không giới hạn, nhưng nên validate |

:::warning Không lưu dữ liệu nhạy cảm
Không lưu mật khẩu, token, hoặc thông tin thẻ tín dụng vào local storage. Chỉ lưu dữ liệu không nhạy cảm hoặc đã mã hóa.
:::

## Best Practices

### 1. Validate file type

```javascript
const chooseImage = () => {
  window.WindVane.call('WVFile', 'chooseFiles', 
    { accept: 'image/jpeg,image/png', count: 1 },
    (result) => {
      const file = result.files[0];
      
      // Kiểm tra kích thước
      if (file.size > 5 * 1024 * 1024) { // 5MB
        alert('File quá lớn. Tối đa 5MB');
        return;
      }
      
      // Kiểm tra định dạng
      if (!['image/jpeg', 'image/png'].includes(file.type)) {
        alert('Chỉ hỗ trợ JPG và PNG');
        return;
      }
      
      uploadFile(file);
    }
  );
};
```

### 2. Retry mechanism cho upload

```javascript
const uploadWithRetry = async (url, filePath, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await windvaneCall('WVFile', 'uploadFile', { url, filePath });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
};
```

### 3. Progress tracking

```javascript
window.WindVane.call('WVFile', 'uploadFile', {
  url: 'https://api.example.com/upload',
  filePath: '/path/to/file'
}, (result) => {
  // Success
  console.log('Upload complete');
}, (error) => {
  // Error
  console.error('Upload failed:', error);
}, (progress) => {
  // Progress callback
  const percent = (progress.loaded / progress.total) * 100;
  updateProgressBar(percent);
});
```

## Xem thêm

- [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) - Ví dụ upload/download
