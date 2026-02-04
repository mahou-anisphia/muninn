---
sidebar_position: 8
---

# setBackgroundColor - Màu nền container

Đặt màu nền cho container của miniapp.

## API Call

```javascript
window.WindVane.call('WVBase', 'setBackgroundColor', {
  color: '#FFFFFF',
  alpha: 1
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `color` | `string` | Có | Màu hex, vd: `#FFFFFF` |
| `alpha` | `number` | Không | Độ trong suốt 0-1, mặc định 1 |

## Use Case: Dark Mode

```javascript
class ThemeManager {
  setTheme(isDark) {
    const bgColor = isDark ? '#1a1a1a' : '#FFFFFF';
    
    window.WindVane.call('WVBase', 'setBackgroundColor', {
      color: bgColor,
      alpha: 1
    });
    
    document.body.className = isDark ? 'dark-theme' : 'light-theme';
  }
}
```
