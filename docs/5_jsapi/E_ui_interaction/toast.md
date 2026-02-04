---
sidebar_position: 2
---

# toast

Hiển thị thông báo toast (notification ngắn gọn) cho user.

## Cú pháp

```javascript
window.WindVane.call(
  'WVUIToast',
  'toast',
  {
    message: string,
    duration: number,
    type: 'success' | 'error' | 'info' | 'warning'
  },
  (result) => { /* success */ },
  (error) => { /* fail */ }
);
```

## Tham số đầu vào

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `message` | `string` | Có | Nội dung thông báo |
| `duration` | `number` | Không | Thời gian hiển thị (ms) - Mặc định: 2000 (2s) |
| `type` | `string` | Không | Loại toast: `'success'`, `'error'`, `'info'`, `'warning'` |

## Ví dụ

### 1. Toast cơ bản

```javascript
window.WindVane.call('WVUIToast', 'toast',
  { message: 'Thao tác thành công!' },
  () => {},
  (error) => console.error(error)
);
```

### 2. Toast với type và duration

```javascript
// Success toast
window.WindVane.call('WVUIToast', 'toast',
  {
    message: 'Lưu thành công!',
    type: 'success',
    duration: 3000
  }
);

// Error toast
window.WindVane.call('WVUIToast', 'toast',
  {
    message: 'Có lỗi xảy ra, vui lòng thử lại',
    type: 'error',
    duration: 3000
  }
);

// Warning toast
window.WindVane.call('WVUIToast', 'toast',
  {
    message: 'Bạn có chắc chắn muốn xóa?',
    type: 'warning',
    duration: 3000
  }
);
```

### 3. Helper function

```javascript
const showToast = (message, type = 'info', duration = 2000) => {
  return new Promise((resolve) => {
    window.WindVane.call('WVUIToast', 'toast',
      { message, type, duration },
      resolve,
      (error) => {
        console.error('Toast error:', error);
        resolve();
      }
    );
  });
};

// Sử dụng
await showToast('Đã sao chép vào clipboard', 'success');
await showToast('Không thể kết nối', 'error', 3000);
```

### 4. Toast manager với queue

```javascript
class ToastManager {
  constructor() {
    this.queue = [];
    this.showing = false;
  }

  async show(message, type = 'info', duration = 2000) {
    this.queue.push({ message, type, duration });
    
    if (!this.showing) {
      await this.processQueue();
    }
  }

  async processQueue() {
    if (this.queue.length === 0) {
      this.showing = false;
      return;
    }

    this.showing = true;
    const { message, type, duration } = this.queue.shift();

    await new Promise((resolve) => {
      window.WindVane.call('WVUIToast', 'toast',
        { message, type, duration },
        resolve,
        resolve
      );
    });

    // Đợi toast hiển thị xong
    await new Promise(r => setTimeout(r, duration + 100));

    // Process toast tiếp theo
    await this.processQueue();
  }

  success(message, duration) {
    return this.show(message, 'success', duration);
  }

  error(message, duration) {
    return this.show(message, 'error', duration);
  }

  warning(message, duration) {
    return this.show(message, 'warning', duration);
  }

  info(message, duration) {
    return this.show(message, 'info', duration);
  }
}

// Global instance
const toast = new ToastManager();

// Sử dụng
toast.success('Lưu thành công!');
toast.error('Có lỗi xảy ra');
toast.warning('Cảnh báo!');
toast.info('Thông tin');
```

### 5. React Hook

```jsx
import { useCallback } from 'react';

function useToast() {
  const showToast = useCallback((message, type = 'info', duration = 2000) => {
    return new Promise((resolve) => {
      window.WindVane.call('WVUIToast', 'toast',
        { message, type, duration },
        resolve,
        (error) => {
          console.error('Toast error:', error);
          resolve();
        }
      );
    });
  }, []);

  const success = useCallback((message, duration) => {
    return showToast(message, 'success', duration);
  }, [showToast]);

  const error = useCallback((message, duration) => {
    return showToast(message, 'error', duration);
  }, [showToast]);

  const warning = useCallback((message, duration) => {
    return showToast(message, 'warning', duration);
  }, [showToast]);

  const info = useCallback((message, duration) => {
    return showToast(message, 'info', duration);
  }, [showToast]);

  return { showToast, success, error, warning, info };
}

// Component sử dụng
function FormComponent() {
  const toast = useToast();

  const handleSubmit = async (data) => {
    try {
      await saveData(data);
      toast.success('Lưu thành công!');
    } catch (err) {
      toast.error('Không thể lưu dữ liệu');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit">Lưu</button>
    </form>
  );
}
```

### 6. Với async operations

```javascript
const saveWithToast = async (data) => {
  try {
    await showToast('Đang lưu...', 'info', 1000);
    
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    await showToast('Lưu thành công!', 'success', 2000);
    return true;
  } catch (error) {
    await showToast('Lỗi: ' + error.message, 'error', 3000);
    return false;
  }
};
```

## Use Cases

| Use Case | Type | Duration | Message |
|----------|------|----------|---------|
| **Save success** | `success` | 2000ms | "Lưu thành công!" |
| **Copy to clipboard** | `success` | 1500ms | "Đã sao chép" |
| **Network error** | `error` | 3000ms | "Không có kết nối mạng" |
| **Validation error** | `warning` | 2500ms | "Vui lòng điền đầy đủ thông tin" |
| **Info** | `info` | 2000ms | "Đang xử lý..." |

## Best Practices

### 1. Duration phù hợp

```javascript
const showToastSmart = (message, type = 'info') => {
  // Tính duration dựa trên độ dài message
  const baseTime = 2000;
  const charTime = 50; // 50ms per character
  const duration = Math.min(baseTime + message.length * charTime, 5000);
  
  return showToast(message, type, duration);
};
```

### 2. Không spam toast

```javascript
let lastToastTime = 0;
const MIN_TOAST_INTERVAL = 500; // 500ms

const showToastThrottled = (message, type, duration) => {
  const now = Date.now();
  if (now - lastToastTime < MIN_TOAST_INTERVAL) {
    console.warn('Toast throttled');
    return;
  }
  
  lastToastTime = now;
  return showToast(message, type, duration);
};
```

### 3. Toast cho form validation

```javascript
const validateForm = (formData) => {
  if (!formData.email) {
    showToast('Vui lòng nhập email', 'warning');
    return false;
  }
  
  if (!formData.phone) {
    showToast('Vui lòng nhập số điện thoại', 'warning');
    return false;
  }
  
  if (formData.phone.length !== 10) {
    showToast('Số điện thoại không hợp lệ', 'error');
    return false;
  }
  
  return true;
};
```

### 4. Toast thay vì alert

```javascript
// ❌ Không nên dùng alert (blocking UI)
alert('Có lỗi xảy ra');

// ✅ Nên dùng toast (non-blocking)
showToast('Có lỗi xảy ra', 'error');

// ❌ Không nên dùng confirm cho thông báo đơn giản
const ok = confirm('Lưu thành công!');

// ✅ Nên dùng toast
showToast('Lưu thành công!', 'success');
```

## Khi nào không nên dùng Toast?

| Tình huống | Nên dùng | Lý do |
|------------|----------|-------|
| Lỗi nghiêm trọng | **Dialog** | Cần user acknowledge |
| Xác nhận hành động | **Confirm Dialog** | Cần user choice |
| Form validation | Toast OK | Quick feedback |
| Success message | Toast OK | Non-blocking |
| Network error | Toast hoặc Dialog | Tùy severity |

## Message Guidelines

:::tip Viết message tốt
- **Ngắn gọn**: Tối đa 2 dòng (< 50 ký tự)
- **Rõ ràng**: "Lưu thành công" > "OK"
- **Actionable**: "Không có mạng. Vui lòng kiểm tra kết nối" > "Lỗi"
- **Positive**: "Email không hợp lệ" > "Email sai"
- **No jargon**: "Không thể kết nối" > "Connection timeout error"
:::

## Lỗi thường gặp

Toast API rất ổn định và hiếm khi lỗi, nhưng có thể xảy ra:

| Tình huống | Nguyên nhân | Xử lý |
|------------|-------------|-------|
| Toast không hiện | WindVane chưa ready | Kiểm tra `window.WindVane` |
| Message bị cắt | Quá dài | Giới hạn < 50 chars |
| Spam toast | Gọi quá nhiều lần | Implement throttling |

## Performance

Toast là lightweight API, nhưng nên:

- ✅ Tránh gọi toast trong loop
- ✅ Throttle khi gọi liên tục
- ✅ Queue toast nếu cần hiển thị nhiều cái
- ❌ Không dùng toast cho debug (dùng console.log)

## Xem thêm

- [Loading](./loading) - Hiển thị loading spinner
- [Dialog](./dialog) - Dialog cho interaction quan trọng
- [Confirm](./confirm) - Confirm dialog
