---
sidebar_position: 1
---

# NavigationBar

Tùy chỉnh thanh điều hướng (Navigation Bar) của miniapp.

:::danger Yêu cầu bắt buộc
Miniapp **PHẢI** hiển thị Navigation Bar với:
- Nút Back (←) - cho phép user quay lại
- Nút Close (✕) - cho phép user đóng miniapp

**KHÔNG ĐƯỢC:**
- Ẩn Navigation Bar hoàn toàn
- Ẩn nút Back hoặc Close
- Thay thế bằng Navigation Bar tự thiết kế

Xem thêm: [Yêu cầu nền tảng](../../yeu_cau_nen_tang/index#1-navigation-bar)
:::

## Cú pháp

```javascript
window.WindVane.call('WVNavigationBar', 'update', params, successCallback, failCallback);
```

## Tham số

### Input

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `title` | `string` | Không | Tiêu đề hiển thị trên thanh |
| `titleColor` | `string` | Không | Màu chữ tiêu đề (hex format: `#RRGGBB`) |
| `backgroundColor` | `string` | Không | Màu nền thanh điều hướng (hex format: `#RRGGBBAA`) |
| `barStyle` | `string` | Không | Kiểu hiển thị: `"normal"` hoặc `"float"` |
| `hideBackButton` | `string` | Không | `"true"` để ẩn nút Back (không khuyến khích) |
| `theme` | `string` | Không | `"light"` hoặc `"dark"` |

:::warning Về hideBackButton
Mặc dù API hỗ trợ `hideBackButton: "true"`, việc ẩn nút Back **không được khuyến khích** vì làm giảm trải nghiệm người dùng. Chỉ sử dụng trong trường hợp đặc biệt (ví dụ: màn hình splash).
:::

### Success Callback

```javascript
{
  success: true
}
```

### Fail Callback

```javascript
{
  error: string,
  errorMessage: string
}
```

## Ví dụ

### Tùy chỉnh cơ bản

```javascript
const setupNavBar = () => {
  window.WindVane.call('WVNavigationBar', 'update', {
    title: 'Trang chủ',
    titleColor: '#000000',
    backgroundColor: '#FFFFFF',
    barStyle: 'normal',
    hideBackButton: 'false'
  }, () => {
    console.log('Navigation bar updated');
  });
};
```

### Theme sáng/tối

```javascript
const setLightTheme = () => {
  window.WindVane.call('WVNavigationBar', 'update', {
    title: 'My App',
    backgroundColor: '#FFFFFF',
    titleColor: '#000000',
    theme: 'light'
  });
};

const setDarkTheme = () => {
  window.WindVane.call('WVNavigationBar', 'update', {
    title: 'My App',
    backgroundColor: '#1A1A1A',
    titleColor: '#FFFFFF',
    theme: 'dark'
  });
};
```

### Ẩn để hiển thị fullscreen (không khuyến khích)

```javascript
const hideNavBar = () => {
  window.WindVane.call('WVNavigationBar', 'update', {
    title: '',
    titleColor: '#00000000',      // Trong suốt
    backgroundColor: '#00000000',  // Trong suốt
    barStyle: 'float',
    hideBackButton: 'true'
  });
};
```

:::caution Chỉ dùng cho splash screen
Ẩn Navigation Bar chỉ nên áp dụng cho màn hình splash hoặc intro. Các màn hình chính **phải** hiển thị Navigation Bar với nút Back/Close.
:::

### Cập nhật động theo route

```javascript
const updateNavBarByRoute = (route) => {
  const navConfig = {
    '/': {
      title: 'Trang chủ',
      backgroundColor: '#EE0033',
      titleColor: '#FFFFFF'
    },
    '/profile': {
      title: 'Hồ sơ',
      backgroundColor: '#FFFFFF',
      titleColor: '#000000'
    },
    '/settings': {
      title: 'Cài đặt',
      backgroundColor: '#F5F5F5',
      titleColor: '#333333'
    }
  };

  const config = navConfig[route] || navConfig['/'];
  
  window.WindVane.call('WVNavigationBar', 'update', {
    ...config,
    barStyle: 'normal',
    hideBackButton: 'false'
  });
};

// Sử dụng với React Router
useEffect(() => {
  updateNavBarByRoute(location.pathname);
}, [location.pathname]);
```

### Tích hợp với React component

```javascript
import { useEffect } from 'react';

const Page = ({ title, theme = 'light' }) => {
  useEffect(() => {
    const isDark = theme === 'dark';
    
    window.WindVane?.call('WVNavigationBar', 'update', {
      title,
      backgroundColor: isDark ? '#000000' : '#FFFFFF',
      titleColor: isDark ? '#FFFFFF' : '#000000',
      barStyle: 'normal',
      hideBackButton: 'false',
      theme
    });
  }, [title, theme]);

  return (
    <div>
      {/* Page content */}
    </div>
  );
};

// Sử dụng
<Page title="Trang chủ" theme="light" />
```

### Tích hợp với Vue component

```vue
<template>
  <div>
    <!-- Page content -->
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    theme: {
      type: String,
      default: 'light'
    }
  },
  watch: {
    title: 'updateNavBar',
    theme: 'updateNavBar'
  },
  mounted() {
    this.updateNavBar();
  },
  methods: {
    updateNavBar() {
      const isDark = this.theme === 'dark';
      
      window.WindVane?.call('WVNavigationBar', 'update', {
        title: this.title,
        backgroundColor: isDark ? '#000000' : '#FFFFFF',
        titleColor: isDark ? '#FFFFFF' : '#000000',
        barStyle: 'normal',
        hideBackButton: 'false',
        theme: this.theme
      });
    }
  }
};
</script>
```

## Bar Styles

### Normal Style

```javascript
// Navigation bar cố định ở đầu màn hình
window.WindVane.call('WVNavigationBar', 'update', {
  title: 'Normal Bar',
  barStyle: 'normal',
  backgroundColor: '#FFFFFF'
});
```

### Float Style

```javascript
// Navigation bar "nổi" trên nội dung (thường dùng cho splash)
window.WindVane.call('WVNavigationBar', 'update', {
  title: '',
  barStyle: 'float',
  backgroundColor: '#00000000'  // Trong suốt
});
```

## Màu sắc

### Sử dụng Viettel branding

```javascript
const VIETTEL_RED = '#EE0033';
const VIETTEL_GRAY = '#44494D';

window.WindVane.call('WVNavigationBar', 'update', {
  title: 'Viettel App',
  backgroundColor: VIETTEL_RED,
  titleColor: '#FFFFFF'
});
```

### Hỗ trợ alpha channel

```javascript
// Format: #RRGGBBAA
window.WindVane.call('WVNavigationBar', 'update', {
  backgroundColor: '#FFFFFF80'  // 50% opacity
});
```

## Best Practices

### 1. Khởi tạo khi app load

```javascript
// Thiết lập NavBar ngay khi miniapp khởi động
window.addEventListener('DOMContentLoaded', () => {
  if (window.WindVane) {
    window.WindVane.call('WVNavigationBar', 'update', {
      title: 'My App',
      backgroundColor: '#FFFFFF',
      titleColor: '#000000',
      barStyle: 'normal',
      hideBackButton: 'false'
    });
  }
});
```

### 2. Wrapper function tái sử dụng

```javascript
const setNavBar = (title, options = {}) => {
  const defaults = {
    title,
    backgroundColor: '#FFFFFF',
    titleColor: '#000000',
    barStyle: 'normal',
    hideBackButton: 'false',
    theme: 'light'
  };

  const config = { ...defaults, ...options };

  if (window.WindVane) {
    window.WindVane.call('WVNavigationBar', 'update', config);
  }
};

// Sử dụng
setNavBar('Trang chủ');
setNavBar('Cài đặt', { backgroundColor: '#EE0033', titleColor: '#FFF' });
```

### 3. Lưu trạng thái theme

```javascript
const ThemeManager = {
  current: 'light',

  set(theme) {
    this.current = theme;
    localStorage.setItem('theme', theme);
    this.apply();
  },

  apply() {
    const isDark = this.current === 'dark';
    window.WindVane?.call('WVNavigationBar', 'update', {
      backgroundColor: isDark ? '#000000' : '#FFFFFF',
      titleColor: isDark ? '#FFFFFF' : '#000000',
      theme: this.current
    });
  },

  init() {
    this.current = localStorage.getItem('theme') || 'light';
    this.apply();
  }
};

// Khởi tạo khi app load
ThemeManager.init();
```

### 4. Responsive title

```javascript
const updateTitle = (fullTitle) => {
  // Rút ngắn title nếu quá dài (trên mobile)
  const maxLength = 20;
  const title = fullTitle.length > maxLength 
    ? fullTitle.substring(0, maxLength) + '...'
    : fullTitle;

  window.WindVane.call('WVNavigationBar', 'update', { title });
};
```

## Lưu ý

:::warning Không ẩn Navigation Bar
Theo yêu cầu của nền tảng Tammi, miniapp **không được** ẩn hoàn toàn Navigation Bar. Nếu cần fullscreen, liên hệ Viettel để được tư vấn.
:::

:::tip Màu tương phản
Đảm bảo màu chữ và màu nền có độ tương phản đủ để dễ đọc (WCAG AA standard: contrast ratio ≥ 4.5:1).

```javascript
// Tốt
{ backgroundColor: '#FFFFFF', titleColor: '#000000' }  // Contrast: 21:1

// Tránh
{ backgroundColor: '#EEEEEE', titleColor: '#CCCCCC' }  // Contrast: 1.6:1 (quá thấp)
```
:::

:::info Platform requirements
Navigation Bar là yêu cầu bắt buộc của nền tảng. Miniapp vi phạm có thể bị từ chối khi review. Xem [Yêu cầu nền tảng](../../yeu_cau_nen_tang/index).
:::

## Xem thêm

- [Yêu cầu nền tảng](../../yeu_cau_nen_tang/index#1-navigation-bar) - Quy định chi tiết về Navigation Bar
- [Toast API](./toast) - Thông báo ngắn
- [Loading API](./loading) - Loading spinner
