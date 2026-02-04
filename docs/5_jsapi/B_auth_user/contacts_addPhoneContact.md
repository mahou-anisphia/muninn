---
sidebar_position: 1
---

# Contacts addPhoneContact - Thêm liên hệ

Mở giao diện thêm liên hệ mới vào danh bạ.

## API Call

```javascript
window.WindVane.call('WVContacts', 'addPhoneContact', {
  firstName: 'Nguyễn',
  lastName: 'Văn A',
  mobilePhoneNumber: '0987654321',
  email: 'nguyenvana@example.com',
  organization: 'Viettel'
}, (result) => {
  console.log('Contact added');
}, (error) => {
  console.error('Failed to add contact:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `firstName` | `string` | Có | Tên |
| `lastName` | `string` | Không | Họ |
| `mobilePhoneNumber` | `string` | Có | Số điện thoại |
| `email` | `string` | Không | Email |
| `organization` | `string` | Không | Công ty/Tổ chức |

## Use Case 1: Save Customer Support Contact

```javascript
class SupportContactSaver {
  saveHotline() {
    window.WindVane.call('WVUIDialog', 'confirm', {
      title: 'Lưu số hotline',
      message: 'Lưu số hotline hỗ trợ vào danh bạ?',
      okbutton: 'Lưu',
      cancelbutton: 'Không'
    }, () => {
      window.WindVane.call('WVContacts', 'addPhoneContact', {
        firstName: 'Hotline',
        lastName: 'Viettel Tammi',
        mobilePhoneNumber: '1800-1234',
        organization: 'Viettel',
        email: 'support@tammi.vn'
      }, () => {
        window.WindVane.call('WVUIToast', 'toast', {
          message: 'Đã lưu vào danh bạ'
        });
      });
    });
  }
}

// Sử dụng
const saver = new SupportContactSaver();
saver.saveHotline();
```

## Use Case 2: Referral Program

```javascript
class ReferralProgram {
  async inviteFriend(friendName, friendPhone) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVContacts', 'addPhoneContact', {
        firstName: friendName,
        lastName: '(Bạn bè)',
        mobilePhoneNumber: friendPhone,
        organization: 'Được giới thiệu qua Tammi'
      }, () => {
        // Send invite SMS
        this.sendInviteSMS(friendPhone);
        resolve(true);
      }, reject);
    });
  }

  sendInviteSMS(phone) {
    // Send SMS with referral link
    window.WindVane.call('WVSMS', 'send', {
      phoneNumber: phone,
      message: 'Tôi mời bạn dùng Tammi! Tải tại: https://tammi.vn/download'
    });
  }
}

// Sử dụng
const referral = new ReferralProgram();
await referral.inviteFriend('Trần Văn B', '0912345678');
```

## Use Case 3: Quick Save from Profile

```javascript
class ProfileContactSaver {
  constructor() {
    this.setupSaveButton();
  }

  setupSaveButton() {
    const saveBtn = document.getElementById('save-contact-btn');
    saveBtn.onclick = () => this.saveProfileContact();
  }

  saveProfileContact() {
    const profile = this.getCurrentProfile();
    
    window.WindVane.call('WVContacts', 'addPhoneContact', {
      firstName: profile.firstName,
      lastName: profile.lastName,
      mobilePhoneNumber: profile.phone,
      email: profile.email,
      organization: profile.company,
      homePhoneNumber: profile.officePhone,
      note: `Đã lưu từ Tammi vào ${new Date().toLocaleDateString()}`
    }, () => {
      this.showSuccessMessage();
    }, (error) => {
      this.showErrorMessage(error);
    });
  }

  getCurrentProfile() {
    return {
      firstName: document.getElementById('first-name').textContent,
      lastName: document.getElementById('last-name').textContent,
      phone: document.getElementById('phone').textContent,
      email: document.getElementById('email').textContent,
      company: document.getElementById('company').textContent,
      officePhone: document.getElementById('office-phone').textContent
    };
  }

  showSuccessMessage() {
    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Đã lưu liên hệ vào danh bạ',
      duration: 2000
    });
  }

  showErrorMessage(error) {
    window.WindVane.call('WVUIDialog', 'alert', {
      title: 'Lỗi',
      message: `Không thể lưu liên hệ: ${error.message}`
    });
  }
}

// Sử dụng
const contactSaver = new ProfileContactSaver();
```

## Best Practices

:::warning Permission Required
Yêu cầu quyền WRITE_CONTACTS. Kiểm tra permission trước khi gọi API.
:::

### 1. Check Permission

```javascript
async function checkContactsPermission() {
  return new Promise((resolve) => {
    window.WindVane.call('WVContacts', 'askAuth', {}, (result) => {
      if (!result.isAuthed) {
        window.WindVane.call('WVUIDialog', 'alert', {
          message: 'Cần quyền truy cập danh bạ để lưu liên hệ'
        });
        resolve(false);
      } else {
        resolve(true);
      }
    });
  });
}

// Sử dụng
if (await checkContactsPermission()) {
  window.WindVane.call('WVContacts', 'addPhoneContact', { /* ... */ });
}
```

### 2. Validate Phone Number

```javascript
function validatePhoneNumber(phone) {
  // Remove spaces and special chars
  const cleaned = phone.replace(/[^\d+]/g, '');
  
  // Vietnam phone number validation
  const vnPhoneRegex = /^(\+84|0)(3|5|7|8|9)\d{8}$/;
  
  return vnPhoneRegex.test(cleaned);
}

function saveContact(name, phone) {
  if (!validatePhoneNumber(phone)) {
    window.WindVane.call('WVUIDialog', 'alert', {
      message: 'Số điện thoại không hợp lệ'
    });
    return;
  }

  window.WindVane.call('WVContacts', 'addPhoneContact', {
    firstName: name,
    mobilePhoneNumber: phone
  });
}
```

### 3. Handle Duplicates

```javascript
async function addContactSafely(contactData) {
  // Check if contact exists
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVContacts', 'find', {
      filter: { phone: contactData.mobilePhoneNumber }
    }, (result) => {
      if (result.contacts && result.contacts.length > 0) {
        // Contact exists, ask user
        window.WindVane.call('WVUIDialog', 'confirm', {
          message: 'Số này đã có trong danh bạ. Vẫn lưu?',
          okbutton: 'Lưu',
          cancelbutton: 'Hủy'
        }, () => {
          window.WindVane.call('WVContacts', 'addPhoneContact', contactData, resolve, reject);
        }, () => reject(new Error('User cancelled')));
      } else {
        // Safe to add
        window.WindVane.call('WVContacts', 'addPhoneContact', contactData, resolve, reject);
      }
    });
  });
}
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `PERMISSION_DENIED` | Không có quyền danh bạ | Gọi `askAuth` trước |
| `INVALID_PHONE` | Số điện thoại không hợp lệ | Validate phone format |
| `USER_CANCELLED` | User hủy thao tác | Xử lý gracefully |

## API liên quan

- [contacts_askAuth](./contacts_askAuth) - Xin quyền danh bạ
- [contacts_find](./contacts_find) - Tìm kiếm liên hệ
- [contacts_choose](./contacts_choose) - Chọn từ danh bạ
