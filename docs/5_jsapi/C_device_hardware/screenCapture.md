---
sidebar_position: 24
---

# screenCapture - Chụp màn hình

Chụp screenshot của màn hình hiện tại.

## API Call

```javascript
window.WindVane.call('WVScreenCapture', 'capture', {
  inAlbum: 'true'
}, (result) => {
  console.log('Screenshot saved:', result.path);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `inAlbum` | `string` | Không | `true` = lưu vào thư viện, `false` = chỉ trả path |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `path` | `string` | Đường dẫn file screenshot |

## Use Case: Share Screenshot

```javascript
async function captureAndShare() {
  window.WindVane.call('WVScreenCapture', 'capture', {
    inAlbum: 'false'
  }, (result) => {
    window.WindVane.call('WVShare', 'shareImage', {
      imagePath: result.path,
      title: 'Chia sẻ screenshot'
    });
  });
}
```
