---
sidebar_position: 21
---

# makePhoneCall - Thực hiện cuộc gọi

Mở màn hình quay số hoặc gọi trực tiếp.

## API Call

```javascript
// Gọi trực tiếp
window.WindVane.call('WVCall', 'call', {
  phone: '0987654321'
});

// Mở màn hình quay số
window.WindVane.call('WVCall', 'dial', {
  phone: '0987654321'
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `phone` | `string` | Có | Số điện thoại |

## Use Case: Customer Support

```javascript
class SupportCaller {
  constructor() {
    this.hotline = '1800-1234';
  }

  callSupport() {
    window.WindVane.call('WVUIDialog', 'confirm', {
      title: 'Gọi hotline',
      message: `Gọi tổng đài ${this.hotline}?`,
      okbutton: 'Gọi',
      cancelbutton: 'Hủy'
    }, () => {
      window.WindVane.call('WVCall', 'call', {
        phone: this.hotline
      });
    });
  }
}

// Sử dụng
const support = new SupportCaller();
support.callSupport();
```

## API liên quan

- [contacts_choose](../B_auth_user/contacts_choose) - Chọn từ danh bạ
