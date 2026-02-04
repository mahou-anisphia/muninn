---
sidebar_position: 7
---

# openSettings - Mở cài đặt

Mở trang cài đặt của ứng dụng trong Settings hệ thống.

## API Call

```javascript
window.WindVane.call('WVApplication', 'openSettings', {
  type: 'Notification'
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `type` | `string` | Không | `Notification`, `Location`, `Camera`, `Bluetooth` |

## Use Case: Permission Guide

```javascript
function guideUserToEnableLocation() {
  window.WindVane.call('WVUIDialog', 'confirm', {
    title: 'Cần quyền vị trí',
    message: 'App cần quyền truy cập vị trí để hiển thị bản đồ',
    okbutton: 'Mở cài đặt',
    cancelbutton: 'Hủy'
  }, () => {
    window.WindVane.call('WVApplication', 'openSettings', {
      type: 'Location'
    });
  });
}
```
