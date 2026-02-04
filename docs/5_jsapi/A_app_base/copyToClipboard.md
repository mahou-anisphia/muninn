---
sidebar_position: 4
---

# copyToClipboard

Sao chép văn bản vào bộ nhớ đệm (clipboard) của thiết bị.

## Cú pháp

```javascript
window.WindVane.call('WVBase', 'copyToClipboard', params, successCallback, failCallback);
```

## Tham số

### Input

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `text` | `string` | Có | Văn bản cần sao chép |

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

### Copy đơn giản

```javascript
const copyText = (text) => {
  window.WindVane.call('WVBase', 'copyToClipboard', 
    { text }, 
    () => {
      alert('Đã sao chép!');
    },
    (error) => {
      alert('Không thể sao chép: ' + error.errorMessage);
    }
  );
};

// Sử dụng
copyText('Hello Tammi!');
```

### Copy mã giới thiệu

```javascript
const shareReferralCode = (code) => {
  window.WindVane.call('WVBase', 'copyToClipboard', 
    { text: code }, 
    () => {
      // Hiển thị toast thay vì alert
      showToast('Đã copy mã giới thiệu: ' + code);
    }
  );
};
```

### Copy với xác nhận

```javascript
const copyWithConfirm = (text, label) => {
  // Hiển thị confirm trước khi copy
  if (confirm(`Copy ${label}?`)) {
    window.WindVane.call('WVBase', 'copyToClipboard', { text }, () => {
      console.log('Copied:', text);
    });
  }
};

// Ví dụ: Copy email
copyWithConfirm('support@tammi.vn', 'email hỗ trợ');
```

### Tích hợp với UI

```javascript
// HTML
// <button onclick="copyPromoCode()">Copy mã khuyến mãi</button>
// <span id="promo-code">TAMMI2024</span>

const copyPromoCode = () => {
  const code = document.getElementById('promo-code').textContent;
  
  window.WindVane.call('WVBase', 'copyToClipboard', 
    { text: code },
    () => {
      // Thay đổi text button để feedback
      const btn = event.target;
      const originalText = btn.textContent;
      btn.textContent = '✓ Đã copy!';
      
      setTimeout(() => {
        btn.textContent = originalText;
      }, 2000);
    }
  );
};
```

## Use Cases

### 1. Share link sản phẩm

```javascript
const shareProduct = (productId) => {
  const url = `https://tammi.vn/products/${productId}`;
  
  window.WindVane.call('WVBase', 'copyToClipboard', { text: url }, () => {
    alert('Đã copy link sản phẩm. Chia sẻ với bạn bè nhé!');
  });
};
```

### 2. Copy thông tin thanh toán

```javascript
const copyPaymentInfo = () => {
  const bankInfo = `
Ngân hàng: Vietcombank
Số TK: 1234567890
Chủ TK: CONG TY TNHH ABC
Nội dung: DH${orderId}
  `.trim();

  window.WindVane.call('WVBase', 'copyToClipboard', { text: bankInfo }, () => {
    showToast('Đã copy thông tin chuyển khoản');
  });
};
```

### 3. Copy kết quả tính toán

```javascript
const copyCalculation = (result) => {
  const text = `Kết quả: ${result.toLocaleString('vi-VN')} VNĐ`;
  
  window.WindVane.call('WVBase', 'copyToClipboard', { text }, () => {
    console.log('Copied result');
  });
};
```

## Best Practices

### 1. Luôn có feedback cho user

```javascript
// ✓ Tốt - có feedback
window.WindVane.call('WVBase', 'copyToClipboard', { text }, () => {
  showToast('Đã copy!'); // User biết copy thành công
});

// ✗ Tránh - không có feedback
window.WindVane.call('WVBase', 'copyToClipboard', { text }, () => {
  // Không làm gì - user không biết đã copy chưa
});
```

### 2. Xử lý lỗi gracefully

```javascript
const safeCopy = (text) => {
  window.WindVane.call('WVBase', 'copyToClipboard', 
    { text },
    () => {
      showToast('✓ Đã copy');
    },
    (error) => {
      // Fallback: hiển thị text để user có thể copy thủ công
      prompt('Không thể tự động copy. Vui lòng copy thủ công:', text);
    }
  );
};
```

### 3. Giới hạn độ dài text

```javascript
const copyLongText = (text) => {
  const MAX_LENGTH = 10000; // 10k chars
  
  if (text.length > MAX_LENGTH) {
    alert('Nội dung quá dài để copy');
    return;
  }

  window.WindVane.call('WVBase', 'copyToClipboard', { text }, () => {
    showToast('Đã copy');
  });
};
```

## Lưu ý

:::warning Không copy dữ liệu nhạy cảm
Tránh copy mật khẩu, mã OTP, hoặc thông tin cá nhân nhạy cảm vào clipboard vì có thể bị các app khác đọc.
:::

:::tip Kết hợp với Toast
Dùng API Toast thay vì `alert()` để feedback tự nhiên hơn:

```javascript
window.WindVane.call('WVUIToast', 'toast', { message: 'Đã copy!' });
```
:::

## Xem thêm

- [Toast API](../E_ui_interaction/toast) - Hiển thị thông báo
- [Share API](../F_media_location/share) - Chia sẻ nội dung
