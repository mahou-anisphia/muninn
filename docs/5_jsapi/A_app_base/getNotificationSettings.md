---
sidebar_position: 3
---

# getNotificationSettings - Cài đặt thông báo

Kiểm tra trạng thái quyền thông báo của ứng dụng.

## API Call

```javascript
window.WindVane.call('WVApplication', 'getNotificationSettings', {}, (result) => {
  console.log('Notification status:', result.status);
});
```

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `status` | `string` | `authorized`, `denied`, `notDetermined` |

## Use Case: Request Notification Permission

```javascript
async function checkNotificationPermission() {
  return new Promise((resolve) => {
    window.WindVane.call('WVApplication', 'getNotificationSettings', {}, (result) => {
      if (result.status === 'denied') {
        window.WindVane.call('WVUIDialog', 'confirm', {
          message: 'Bật thông báo để nhận tin tức mới nhất',
          okbutton: 'Mở cài đặt',
          cancelbutton: 'Để sau'
        }, () => {
          window.WindVane.call('WVApplication', 'openSettings', {
            type: 'Notification'
          });
        });
      }
      resolve(result.status);
    });
  });
}
```
