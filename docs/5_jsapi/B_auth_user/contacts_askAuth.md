---
sidebar_position: 2
---

# Contacts askAuth - Xin quyền danh bạ

Yêu cầu quyền truy cập danh bạ từ người dùng.

## API Call

```javascript
window.WindVane.call('WVContacts', 'askAuth', {}, (result) => {
  console.log('Is authorized:', result.isAuthed);
});
```

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `isAuthed` | `boolean` | `true` nếu có quyền |

## Use Case: Permission Flow

```javascript
class ContactsPermissionManager {
  async requestPermission() {
    return new Promise((resolve) => {
      window.WindVane.call('WVContacts', 'askAuth', {}, (result) => {
        if (result.isAuthed) {
          resolve(true);
        } else {
          this.showPermissionDeniedDialog();
          resolve(false);
        }
      });
    });
  }

  showPermissionDeniedDialog() {
    window.WindVane.call('WVUIDialog', 'confirm', {
      title: 'Cần quyền danh bạ',
      message: 'App cần quyền truy cập danh bạ để lưu liên hệ',
      okbutton: 'Mở cài đặt',
      cancelbutton: 'Hủy'
    }, () => {
      window.WindVane.call('WVApplication', 'openSettings', {
        type: 'Contacts'
      });
    });
  }

  async ensurePermission() {
    const hasPermission = await this.requestPermission();
    
    if (!hasPermission) {
      throw new Error('Contacts permission denied');
    }
    
    return true;
  }
}

// Sử dụng
const permMgr = new ContactsPermissionManager();

try {
  await permMgr.ensurePermission();
  // Proceed with contacts operation
  window.WindVane.call('WVContacts', 'choose', {}, handleContact);
} catch (error) {
  console.log('User denied permission');
}
```

## API liên quan

- [contacts_addPhoneContact](./contacts_addPhoneContact) - Thêm liên hệ
- [contacts_choose](./contacts_choose) - Chọn từ danh bạ
