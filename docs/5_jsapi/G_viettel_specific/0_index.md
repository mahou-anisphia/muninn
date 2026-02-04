---
sidebar_position: 0
id: index
---

# Viettel Specific APIs

Các API đặc thù của nền tảng Tammi Viettel.

## Danh sách APIs

### Device Service
- **addToWhiteList** - Thêm miniapp vào danh sách trắng của thiết bị

### Development Tools
- **clearAuthCache** - Xóa cache xác thực (môi trường Dev)

:::info Chỉ dùng khi cần thiết
Các API này là đặc thù của Viettel và thường chỉ được sử dụng trong các trường hợp đặc biệt hoặc môi trường development.
:::

## Use Cases

### addToWhiteList

Thêm miniapp vào danh sách trắng để bỏ qua một số giới hạn (ví dụ: rate limit).

```javascript
const addToWhiteList = (miniAppId) => {
  window.WindVane.call('VTDeviceService', 'addToWhiteList', 
    { miniAppId },
    () => {
      console.log('Added to whitelist');
    },
    (error) => {
      console.error('Failed to add to whitelist:', error);
    }
  );
};

// Sử dụng
addToWhiteList('your-miniapp-id');
```

:::warning Yêu cầu phê duyệt
API này yêu cầu phê duyệt đặc biệt từ Viettel. Không thể tự sử dụng mà không được cấp quyền.
:::

### clearAuthCache (Dev only)

Xóa cache xác thực - hữu ích khi test luồng SSO.

```javascript
const clearAuthCache = (appId) => {
  // CHỈ DÙNG TRONG MÔI TRƯỜNG DEV
  if (process.env.NODE_ENV !== 'production') {
    window.WindVane.call('ViettelDevServices', 'clearCache', 
      { appId },
      () => {
        console.log('Auth cache cleared - reload to re-authenticate');
        window.location.reload();
      }
    );
  }
};
```

:::danger Không dùng trong production
API này chỉ hoạt động trong môi trường development/staging. Sẽ bị vô hiệu hóa trong production build.
:::

## Best Practices

### 1. Environment check

```javascript
const isDev = () => {
  return process.env.NODE_ENV === 'development' || 
         window.location.hostname === 'localhost';
};

const clearCache = () => {
  if (isDev()) {
    window.WindVane.call('ViettelDevServices', 'clearCache', { ... });
  } else {
    console.warn('clearCache is only available in dev environment');
  }
};
```

### 2. Feature flags

```javascript
const FEATURES = {
  whiteList: false,  // Enable khi được Viettel approve
  devTools: isDev()
};

const useViettelAPI = (feature, callback) => {
  if (FEATURES[feature]) {
    callback();
  } else {
    console.warn(`Feature ${feature} is not enabled`);
  }
};

// Sử dụng
useViettelAPI('whiteList', () => {
  addToWhiteList('my-app-id');
});
```

## Lưu ý

:::info Liên hệ Viettel
Nếu cần sử dụng các API này, vui lòng liên hệ đội ngũ Viettel để được tư vấn và cấp quyền.
:::

## Xem thêm

- [Quyền hạn](../1_truoc_khi_bat_dau) - Hiểu về permissions
- [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) - Ví dụ dev tools
