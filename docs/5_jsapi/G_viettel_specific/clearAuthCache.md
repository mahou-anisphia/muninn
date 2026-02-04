---
sidebar_position: 2
---

# clearAuthCache - Xóa cache xác thực (Dev)

Xóa cache xác thực SSO trong môi trường development.

## API Call

```javascript
window.WindVane.call('ViettelDevServices', 'clearCache', {
  appId: 'your-app-id'
});
```

:::danger Development Only
API này CHỈ hoạt động trong môi trường Dev, không hoạt động trên production.
:::
