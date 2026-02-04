---
sidebar_position: 1
---

# alert - Hộp thoại thông báo

Hiển thị hộp thoại alert với 1 nút OK.

## API Call

```javascript
window.WindVane.call('WVUIDialog', 'alert', {
  title: 'Thông báo',
  message: 'Đăng nhập thành công!',
  buttonText: 'Đồng ý'
}, () => {
  console.log('User clicked OK');
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `title` | `string` | Không | Tiêu đề |
| `message` | `string` | Có | Nội dung thông báo |
| `buttonText` | `string` | Không | Text nút OK, mặc định "OK" |

## Use Case: Error Handler

```javascript
function showError(message) {
  window.WindVane.call('WVUIDialog', 'alert', {
    title: 'Lỗi',
    message: message,
    buttonText: 'Đã hiểu'
  });
}

// Sử dụng
try {
  await someAsyncOperation();
} catch (error) {
  showError(error.message);
}
```
