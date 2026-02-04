---
sidebar_position: 2
---

# canIUse

Kiểm tra tính khả dụng của một API hoặc method cụ thể trước khi sử dụng.

## Cú pháp

```javascript
window.WindVane.call('WVBase', 'canIUse', params, successCallback, failCallback);
```

## Tham số

### Input

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `api` | `string` | Có | Tên API hoặc method cần kiểm tra |

### Success Callback

```javascript
{
  result: boolean  // true nếu API khả dụng, false nếu không
}
```

### Fail Callback

```javascript
{
  error: string,      // Mã lỗi
  errorMessage: string // Mô tả lỗi
}
```

## Ví dụ

### Kiểm tra trước khi gọi API

```javascript
const checkAndScan = () => {
  window.WindVane.call('WVBase', 'canIUse', 
    { api: 'scan' }, 
    (result) => {
      if (result.result) {
        // API khả dụng - tiếp tục quét QR
        window.WindVane.call('WVScan', 'scan', {}, onScanSuccess);
      } else {
        alert('Thiết bị không hỗ trợ quét mã QR');
      }
    },
    (error) => {
      console.error('Lỗi kiểm tra API:', error);
    }
  );
};
```

### Kiểm tra nhiều APIs

```javascript
const checkAPIs = () => {
  const apisToCheck = ['takePhoto', 'getLocation', 'scan'];
  const results = {};

  apisToCheck.forEach(api => {
    window.WindVane.call('WVBase', 'canIUse', { api }, (result) => {
      results[api] = result.result;
      
      if (Object.keys(results).length === apisToCheck.length) {
        console.log('Kết quả kiểm tra:', results);
        // { takePhoto: true, getLocation: false, scan: true }
      }
    });
  });
};
```

### Hiển thị tính năng dựa trên khả dụng

```javascript
const setupFeatures = () => {
  // Kiểm tra Camera
  window.WindVane.call('WVBase', 'canIUse', { api: 'takePhoto' }, (result) => {
    document.getElementById('camera-button').disabled = !result.result;
  });

  // Kiểm tra GPS
  window.WindVane.call('WVBase', 'canIUse', { api: 'getLocation' }, (result) => {
    document.getElementById('location-button').disabled = !result.result;
  });
};
```

## Best Practices

### 1. Kiểm tra ngay khi app khởi động

```javascript
const checkCriticalAPIs = () => {
  const critical = ['getAuthCode', 'takePhoto'];
  
  critical.forEach(api => {
    window.WindVane.call('WVBase', 'canIUse', { api }, (result) => {
      if (!result.result) {
        console.warn(`API ${api} không khả dụng - app có thể bị hạn chế chức năng`);
      }
    });
  });
};

// Gọi khi app khởi động
window.addEventListener('DOMContentLoaded', checkCriticalAPIs);
```

### 2. Tạo wrapper function

```javascript
// Tạo helper function tái sử dụng
const useAPI = (module, method, params) => {
  return new Promise((resolve, reject) => {
    // Bước 1: Kiểm tra quyền
    window.WindVane.call('WVBase', 'canIUse', { api: method }, (canUse) => {
      if (!canUse.result) {
        reject(new Error(`API ${method} không khả dụng`));
        return;
      }

      // Bước 2: Gọi API
      window.WindVane.call(module, method, params, resolve, reject);
    });
  });
};

// Sử dụng
useAPI('WVCamera', 'takePhoto', {})
  .then(result => console.log('Ảnh:', result.url))
  .catch(error => alert('Không thể chụp ảnh: ' + error.message));
```

### 3. Cache kết quả để tăng hiệu năng

```javascript
const apiCache = {};

const canUseAPI = (api) => {
  return new Promise((resolve) => {
    // Kiểm tra cache
    if (apiCache.hasOwnProperty(api)) {
      resolve(apiCache[api]);
      return;
    }

    // Gọi API và cache kết quả
    window.WindVane.call('WVBase', 'canIUse', { api }, (result) => {
      apiCache[api] = result.result;
      resolve(result.result);
    });
  });
};

// Sử dụng
canUseAPI('takePhoto').then(canUse => {
  if (canUse) { /* ... */ }
});
```

## Lưu ý

:::tip Kiểm tra thường xuyên
Quyền có thể thay đổi theo phiên bản miniapp hoặc cấu hình Superapp. Nên kiểm tra mỗi lần trước khi gọi API quan trọng.
:::

:::warning Không thay thế xử lý lỗi
`canIUse` chỉ kiểm tra **khả năng gọi** API. Vẫn cần xử lý lỗi trong callback vì có thể có lỗi khác (user từ chối, thiết bị lỗi...).
:::

## Xem thêm

- [Quyền hạn và phê duyệt](../1_truoc_khi_bat_dau)
- [Xử lý lỗi thường gặp](../1_truoc_khi_bat_dau#xử-lý-lỗi-thường-gặp)
