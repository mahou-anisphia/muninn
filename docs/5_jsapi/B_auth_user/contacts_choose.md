---
sidebar_position: 3
---

# Contacts choose - Chọn liên hệ từ danh bạ

Mở danh bạ để user chọn một số điện thoại.

## API Call

```javascript
window.WindVane.call('WVContacts', 'choose', {}, (result) => {
  console.log('Selected:', result.name, result.phone);
});
```

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `name` | `string` | Tên liên hệ |
| `phone` | `string` | Số điện thoại |

## Use Case: Send Money

```javascript
async function selectRecipient() {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVContacts', 'choose', {}, (result) => {
      resolve({
        name: result.name,
        phone: result.phone.replace(/[^0-9]/g, '') // Remove formatting
      });
    }, reject);
  });
}

// Sử dụng
const recipient = await selectRecipient();
document.getElementById('recipient-name').value = recipient.name;
document.getElementById('recipient-phone').value = recipient.phone;
```
