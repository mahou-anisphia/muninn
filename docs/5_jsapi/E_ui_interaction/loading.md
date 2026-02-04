---
sidebar_position: 5
---

# loading - Hiển thị Loading

Hiển thị/ẩn loading spinner toàn màn hình.

## API Call

```javascript
// Show loading
window.WindVane.call('WVUI', 'showLoadingBox', {
  text: 'Đang tải...'
});

// Hide loading
window.WindVane.call('WVUI', 'hideLoadingBox', {});
```

## Tham số (showLoadingBox)

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `text` | `string` | Không | Text hiển thị dưới spinner |
| `mask` | `boolean` | Không | Chặn tương tác, mặc định true |

## Use Case: Async Operations

```javascript
class LoadingHelper {
  static async execute(asyncFn, loadingText = 'Đang xử lý...') {
    window.WindVane.call('WVUI', 'showLoadingBox', {
      text: loadingText
    });

    try {
      const result = await asyncFn();
      return result;
    } finally {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
    }
  }
}

// Sử dụng
await LoadingHelper.execute(async () => {
  const data = await fetch('/api/data').then(r => r.json());
  processData(data);
}, 'Đang tải dữ liệu...');
```
