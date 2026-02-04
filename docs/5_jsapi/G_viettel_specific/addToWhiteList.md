---
sidebar_position: 1
---

# addToWhiteList - Thêm vào danh sách trắng

Thêm miniapp vào whitelist của Viettel Device Service.

## API Call

```javascript
window.WindVane.call('VTDeviceService', 'addToWhiteList', {
  miniAppId: 'your-app-id'
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `miniAppId` | `string` | Có | ID của miniapp |

:::warning Internal API
API này chỉ dành cho internal use, không public.
:::
