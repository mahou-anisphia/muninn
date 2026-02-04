---
sidebar_position: 6
---

# actionSheet - Menu lựa chọn

Hiển thị menu lựa chọn từ dưới lên.

## API Call

```javascript
window.WindVane.call('WVUIActionSheet', 'show', {
  title: 'Chọn hành động',
  buttons: ['Chụp ảnh', 'Chọn từ thư viện', 'Xem ảnh'],
  cancelButtonText: 'Hủy',
  destructiveButtonIndex: 2
}, (result) => {
  console.log('Selected index:', result.index);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `title` | `string` | Không | Tiêu đề menu |
| `buttons` | `string[]` | Có | Danh sách lựa chọn |
| `cancelButtonText` | `string` | Không | Text nút hủy |
| `destructiveButtonIndex` | `number` | Không | Index nút đỏ (cảnh báo) |

## Use Case: Image Picker

```javascript
function showImagePicker() {
  window.WindVane.call('WVUIActionSheet', 'show', {
    title: 'Chọn ảnh',
    buttons: ['Chụp ảnh mới', 'Chọn từ thư viện'],
    cancelButtonText: 'Hủy'
  }, (result) => {
    if (result.index === 0) {
      window.WindVane.call('WVCamera', 'takePhoto', { mode: 'camera' });
    } else if (result.index === 1) {
      window.WindVane.call('WVCamera', 'takePhoto', { mode: 'album' });
    }
  });
}
```
