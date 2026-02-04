---
sidebar_position: 4
---

# Contacts find - Tìm kiếm liên hệ

Tìm kiếm liên hệ trong danh bạ theo filter.

## API Call

```javascript
window.WindVane.call('WVContacts', 'find', {
  filter: { name: 'Nguyễn' }
}, (result) => {
  console.log('Found contacts:', result.contacts);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `filter` | `object` | Có | Filter: `{ name: string }` hoặc `{ phone: string }` |

## Success Result

```javascript
{
  contacts: [
    {
      id: '123',
      name: 'Nguyễn Văn A',
      phone: '0987654321',
      email: 'nguyenvana@example.com'
    }
  ]
}
```

## Use Case: Contact Search

```javascript
class ContactSearcher {
  async searchByName(name) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVContacts', 'find', {
        filter: { name: name }
      }, (result) => {
        resolve(result.contacts || []);
      }, reject);
    });
  }

  async searchByPhone(phone) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVContacts', 'find', {
        filter: { phone: phone }
      }, (result) => {
        resolve(result.contacts || []);
      }, reject);
    });
  }

  async displaySearchResults(query) {
    const contacts = await this.searchByName(query);
    
    const container = document.getElementById('search-results');
    container.innerHTML = contacts.map(c => `
      <div class="contact-item">
        <div class="name">${c.name}</div>
        <div class="phone">${c.phone}</div>
        <button onclick="selectContact('${c.id}')">Chọn</button>
      </div>
    `).join('');
  }
}

// Sử dụng
const searcher = new ContactSearcher();

document.getElementById('search-input').addEventListener('input', (e) => {
  const query = e.target.value;
  if (query.length >= 2) {
    searcher.displaySearchResults(query);
  }
});
```

## API liên quan

- [contacts_choose](./contacts_choose) - Chọn từ danh bạ
- [contacts_addPhoneContact](./contacts_addPhoneContact) - Thêm liên hệ
