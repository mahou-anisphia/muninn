---
sidebar_position: 3
---

# prompt - Hộp thoại nhập liệu

Hiển thị hộp thoại với text input.

## API Call

```javascript
window.WindVane.call('WVUIDialog', 'prompt', {
  title: 'Nhập tên',
  message: 'Vui lòng nhập tên của bạn',
  placeholder: 'Tên...',
  okbutton: 'Xác nhận',
  cancelbutton: 'Hủy'
}, (result) => {
  console.log('User entered:', result.text);
}, () => {
  console.log('User cancelled');
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `title` | `string` | Không | Tiêu đề |
| `message` | `string` | Có | Nội dung mô tả |
| `placeholder` | `string` | Không | Placeholder cho input |
| `okbutton` | `string` | Không | Text nút OK |
| `cancelbutton` | `string` | Không | Text nút Cancel |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `text` | `string` | Giá trị user nhập |

## Use Case: Name Input

```javascript
async function askUserName() {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVUIDialog', 'prompt', {
      title: 'Tên hiển thị',
      message: 'Nhập tên hiển thị của bạn',
      placeholder: 'VD: Nguyễn Văn A',
      okbutton: 'Xác nhận',
      cancelbutton: 'Bỏ qua'
    }, (result) => {
      if (result.text && result.text.trim()) {
        resolve(result.text.trim());
      } else {
        reject(new Error('Tên không hợp lệ'));
      }
    }, () => reject(new Error('User cancelled')));
  });
}

// Sử dụng
try {
  const name = await askUserName();
  await updateProfile({ displayName: name });
} catch (error) {
  console.log('User skipped name input');
}
```
