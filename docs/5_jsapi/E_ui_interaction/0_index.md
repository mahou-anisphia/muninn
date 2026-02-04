---
sidebar_position: 0
id: index
---

# UI & Interaction APIs

Các API để tùy chỉnh giao diện và tương tác với người dùng.

## Danh sách APIs

| API | Mô tả | Quyền yêu cầu |
|-----|-------|---------------|
| [NavigationBar](./navigationBar) | Tùy chỉnh thanh điều hướng | Không |
| [Toast](./toast) | Hiển thị thông báo ngắn | Không |
| [Loading](./loading) | Hiển thị/ẩn loading spinner | Không |
| [Dialog](./dialog) | Alert, Confirm, Prompt | Không |
| [ActionSheet](./actionSheet) | Menu lựa chọn từ dưới lên | Không |

## Use Cases phổ biến

### 1. Tùy chỉnh Navigation Bar

```javascript
const updateNavBar = (title, theme = 'light') => {
  window.WindVane.call('WVNavigationBar', 'update', {
    title,
    backgroundColor: theme === 'dark' ? '#000000' : '#FFFFFF',
    titleColor: theme === 'dark' ? '#FFFFFF' : '#000000',
    barStyle: 'normal',
    hideBackButton: 'false'
  });
};

// Sử dụng
updateNavBar('Trang chủ', 'light');
```

### 2. Hiển thị Toast notification

```javascript
const showToast = (message, duration = 'short') => {
  window.WindVane.call('WVUIToast', 'toast', {
    message,
    duration  // 'short' (2s) hoặc 'long' (3.5s)
  });
};

// Sử dụng
showToast('Đã lưu thành công');
```

### 3. Loading spinner

```javascript
const showLoading = () => {
  window.WindVane.call('WVUI', 'showLoadingBox', {});
};

const hideLoading = () => {
  window.WindVane.call('WVUI', 'hideLoadingBox', {});
};

// Sử dụng
const fetchData = async () => {
  showLoading();
  try {
    const data = await fetch('/api/data').then(r => r.json());
    return data;
  } finally {
    hideLoading();
  }
};
```

### 4. Confirm dialog

```javascript
const confirmDelete = (itemName, onConfirm) => {
  window.WindVane.call('WVUIDialog', 'confirm', {
    title: 'Xác nhận',
    message: `Bạn có chắc muốn xóa ${itemName}?`,
    okbutton: 'Xóa',
    cancelbutton: 'Hủy'
  }, onConfirm);
};

// Sử dụng
confirmDelete('sản phẩm này', () => {
  deleteItem();
  showToast('Đã xóa');
});
```

### 5. ActionSheet menu

```javascript
const showShareOptions = () => {
  window.WindVane.call('WVUIActionSheet', 'show', {
    title: 'Chia sẻ',
    buttons: ['Facebook', 'Zalo', 'Copy link'],
    cancelButton: 'Hủy'
  }, (result) => {
    switch(result.index) {
      case 0: shareToFacebook(); break;
      case 1: shareToZalo(); break;
      case 2: copyLink(); break;
    }
  });
};
```

## Nguyên tắc thiết kế UI

### 1. Nhất quán với Tammi

Khi tùy chỉnh UI, cần đảm bảo:

- **Màu sắc**: Tuân thủ bảng màu Viettel (đỏ #EE0033 là primary)
- **Typography**: Font chữ và kích thước phù hợp
- **Spacing**: Khoảng cách hợp lý, dễ nhìn trên mobile

### 2. Accessibility

```javascript
// Tốt - màu tương phản rõ ràng
updateNavBar('Trang chủ', {
  backgroundColor: '#FFFFFF',
  titleColor: '#000000'  // Contrast ratio tốt
});

// Tránh - màu tương phản kém
updateNavBar('Trang chủ', {
  backgroundColor: '#EEEEEE',
  titleColor: '#CCCCCC'  // Khó đọc
});
```

### 3. Feedback rõ ràng

```javascript
const saveData = async () => {
  showLoading();
  try {
    await api.save();
    hideLoading();
    showToast('✓ Đã lưu');  // Success feedback
  } catch (error) {
    hideLoading();
    showDialog('Lỗi', error.message);  // Error feedback
  }
};
```

## Yêu cầu bắt buộc

:::danger Navigation Bar bắt buộc
Miniapp **PHẢI** hiển thị Navigation Bar với nút Back (←) và nút Close (✕). Xem chi tiết tại [NavigationBar API](./navigationBar).
:::

:::tip Sử dụng native UI
Ưu tiên sử dụng native UI components (Toast, Dialog) thay vì tự implement bằng HTML/CSS để đảm bảo trải nghiệm nhất quán.
:::

## Best Practices

### 1. Quản lý loading state

```javascript
class LoadingManager {
  constructor() {
    this.count = 0;
  }

  show() {
    if (this.count === 0) {
      window.WindVane.call('WVUI', 'showLoadingBox', {});
    }
    this.count++;
  }

  hide() {
    this.count = Math.max(0, this.count - 1);
    if (this.count === 0) {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
    }
  }
}

const loading = new LoadingManager();

// Sử dụng
loading.show();  // Hiển thị loading
// ... async operation
loading.hide();  // Ẩn loading
```

### 2. Toast queue

```javascript
const toastQueue = [];
let isShowingToast = false;

const queueToast = (message) => {
  toastQueue.push(message);
  processToastQueue();
};

const processToastQueue = () => {
  if (isShowingToast || toastQueue.length === 0) return;

  isShowingToast = true;
  const message = toastQueue.shift();

  window.WindVane.call('WVUIToast', 'toast', { message }, () => {
    setTimeout(() => {
      isShowingToast = false;
      processToastQueue();
    }, 2000);
  });
};
```

### 3. Themed navigation

```javascript
// Lưu theme preference
const setTheme = (isDark) => {
  localStorage.setItem('theme', isDark ? 'dark' : 'light');
  applyTheme(isDark);
};

const applyTheme = (isDark) => {
  const bgColor = isDark ? '#1A1A1A' : '#FFFFFF';
  const textColor = isDark ? '#FFFFFF' : '#000000';

  window.WindVane.call('WVNavigationBar', 'update', {
    backgroundColor: bgColor,
    titleColor: textColor,
    barStyle: isDark ? 'float' : 'normal'
  });
};

// Áp dụng theme khi app khởi động
window.addEventListener('DOMContentLoaded', () => {
  const savedTheme = localStorage.getItem('theme');
  applyTheme(savedTheme === 'dark');
});
```

## Xem thêm

- [Yêu cầu nền tảng](../../yeu_cau_nen_tang/index#1-navigation-bar) - Quy định về Navigation Bar
- [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) - Ví dụ thực tế
