---
sidebar_position: 23
---

# setBrightness - Độ sáng màn hình

Điều chỉnh độ sáng màn hình.

## API Call

```javascript
window.WindVane.call('WVScreen', 'setScreenBrightness', {
  brightness: 0.8
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `brightness` | `number` | Có | Giá trị 0-1 (0 = tối nhất, 1 = sáng nhất) |

## Use Case: Reading Mode

```javascript
class ReadingMode {
  enable() {
    // Giảm độ sáng khi đọc ban đêm
    window.WindVane.call('WVScreen', 'setScreenBrightness', {
      brightness: 0.3
    });
    
    document.body.classList.add('reading-mode');
  }

  disable() {
    // Khôi phục độ sáng
    window.WindVane.call('WVScreen', 'setScreenBrightness', {
      brightness: 0.8
    });
    
    document.body.classList.remove('reading-mode');
  }
}
```
