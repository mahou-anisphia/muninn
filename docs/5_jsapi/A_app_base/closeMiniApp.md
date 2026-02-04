---
sidebar_position: 2
---

# closeMiniApp - Đóng miniapp

Đóng miniapp và quay về SuperApp.

## API Call

```javascript
window.WindVane.call('WVMiniApp', 'close', {});
```

## Use Case: Logout

```javascript
async function logout() {
  await clearUserSession();
  
  window.WindVane.call('WVUIToast', 'toast', {
    message: 'Đã đăng xuất'
  });
  
  setTimeout(() => {
    window.WindVane.call('WVMiniApp', 'close', {});
  }, 1000);
}
```
