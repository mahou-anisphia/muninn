---
sidebar_position: 2
---

# confirm - Hộp thoại xác nhận

Hiển thị hộp thoại với 2 nút: OK và Cancel.

## API Call

```javascript
window.WindVane.call('WVUIDialog', 'confirm', {
  title: 'Xác nhận',
  message: 'Bạn có chắc muốn xóa?',
  okbutton: 'Xóa',
  cancelbutton: 'Hủy'
}, () => {
  console.log('User confirmed');
  deleteItem();
}, () => {
  console.log('User cancelled');
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `title` | `string` | Không | Tiêu đề |
| `message` | `string` | Có | Nội dung |
| `okbutton` | `string` | Không | Text nút OK |
| `cancelbutton` | `string` | Không | Text nút Cancel |

## Use Case: Delete Confirmation

```javascript
async function confirmDelete(itemId) {
  return new Promise((resolve) => {
    window.WindVane.call('WVUIDialog', 'confirm', {
      title: 'Xác nhận xóa',
      message: 'Hành động này không thể hoàn tác',
      okbutton: 'Xóa',
      cancelbutton: 'Hủy'
    }, () => resolve(true), () => resolve(false));
  });
}

// Sử dụng
if (await confirmDelete(itemId)) {
  await deleteItem(itemId);
}
```
