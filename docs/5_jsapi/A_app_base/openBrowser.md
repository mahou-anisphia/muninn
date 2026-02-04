---
sidebar_position: 6
---

# openBrowser - Mở browser

Mở URL bằng trình duyệt mặc định của thiết bị.

## API Call

```javascript
window.WindVane.call('WVBase', 'openBrowser', {
  url: 'https://viettel.vn'
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `url` | `string` | Có | URL cần mở |

## Use Case: Terms & Privacy

```javascript
function openTerms() {
  window.WindVane.call('WVBase', 'openBrowser', {
    url: 'https://example.com/terms'
  });
}

function openPrivacy() {
  window.WindVane.call('WVBase', 'openBrowser', {
    url: 'https://example.com/privacy'
  });
}
```
