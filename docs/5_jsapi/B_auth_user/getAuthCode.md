---
sidebar_position: 1
---

# getAuthCode

Lấy authorization code từ Tammi SSO để thực hiện đăng nhập người dùng.

:::danger API đặc biệt
API này chỉ hoạt động trong môi trường **production** (miniapp đã deploy). Không hoạt động ở local preview.
:::

## Cú pháp

```javascript
window.WindVane.call('wv', 'getAuthCode', params, successCallback, failCallback);
```

## Tham số

### Input

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `appId` | `string` | Có | ID của miniapp (được cấp khi đăng ký) |
| `scopes` | `string[]` | Có | Danh sách quyền yêu cầu |

### Scopes hỗ trợ

| Scope | Dữ liệu nhận được | Cần user consent? |
|-------|-------------------|-------------------|
| `auth_base` | Chỉ authCode, không có thông tin cá nhân | Không |
| `auth_user` | Số điện thoại, họ tên | Có - hiển thị popup xin quyền |

### Success Callback

```javascript
{
  authCode: string  // Authorization code để đổi lấy access token
}
```

### Fail Callback

```javascript
{
  error: string,           // Mã lỗi
  errorMessage: string     // Mô tả lỗi
}
```

## Ví dụ

### Silent Login (auth_base)

```javascript
const silentLogin = () => {
  window.WindVane.call('wv', 'getAuthCode', 
    { 
      appId: 'YOUR_APP_ID',
      scopes: ['auth_base'] 
    },
    (result) => {
      // Gửi authCode lên backend
      fetch('/api/auth/silent-login', {
        method: 'POST',
        body: JSON.stringify({ authCode: result.authCode })
      })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          console.log('Đăng nhập tự động thành công');
        }
      });
    },
    (error) => {
      console.error('Silent login failed:', error);
    }
  );
};
```

### Explicit Login (auth_user)

```javascript
const userLogin = () => {
  window.WindVane.call('wv', 'getAuthCode', 
    { 
      appId: 'YOUR_APP_ID',
      scopes: ['auth_user']  // Yêu cầu consent từ user
    },
    (result) => {
      // User đã đồng ý - gửi authCode lên backend
      fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ authCode: result.authCode })
      })
      .then(res => res.json())
      .then(data => {
        // Lưu token
        localStorage.setItem('token', data.token);
        localStorage.setItem('user', JSON.stringify(data.user));
        
        // Redirect đến dashboard
        window.location.href = '/dashboard';
      });
    },
    (error) => {
      if (error.error === 'USER_DENIED') {
        alert('Bạn cần cấp quyền để sử dụng ứng dụng');
      }
    }
  );
};
```

### Promise Wrapper

```javascript
const getAuthCode = (appId, scopes = ['auth_user']) => {
  return new Promise((resolve, reject) => {
    if (!window.WindVane) {
      reject(new Error('WindVane không khả dụng. Vui lòng chạy trong Tammi app.'));
      return;
    }

    window.WindVane.call('wv', 'getAuthCode',
      { appId, scopes },
      (result) => {
        if (result?.authCode) {
          resolve(result.authCode);
        } else {
          reject(new Error('Không nhận được authCode'));
        }
      },
      (error) => {
        reject(new Error(error.errorMessage || 'Lỗi lấy authCode'));
      }
    );
  });
};

// Sử dụng
try {
  const code = await getAuthCode('YOUR_APP_ID');
  console.log('Auth code:', code);
} catch (error) {
  console.error('Error:', error.message);
}
```

## Luồng xử lý hoàn chỉnh

### Frontend (Miniapp)

```javascript
const performSSO = async () => {
  try {
    // Bước 1: Lấy authCode từ Tammi
    const authCode = await getAuthCode('YOUR_APP_ID', ['auth_user']);
    
    // Bước 2: Gửi authCode lên backend
    const response = await fetch('/api/auth/sso', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ authCode })
    });

    if (!response.ok) {
      throw new Error('Backend authentication failed');
    }

    const { token, user } = await response.json();

    // Bước 3: Lưu token và user info
    localStorage.setItem('authToken', token);
    localStorage.setItem('userInfo', JSON.stringify(user));

    // Bước 4: Redirect hoặc reload app
    window.location.reload();

  } catch (error) {
    console.error('SSO failed:', error);
    // Fallback: hiển thị form đăng nhập thủ công
    showManualLoginForm();
  }
};
```

### Backend Processing

Backend cần thực hiện các bước sau khi nhận authCode:

```javascript
// 1. Đổi authCode lấy access token
POST https://api.tammi.vn/api/v1/auth/oauth/get-access-token
Authorization: Basic {base64(CLIENT_ID:CLIENT_SECRET)}
Body: { authCode: "...", scopes: ["auth_user"] }

// Response:
{
  access_token: "eyJhbGci...",
  expires_in: 432000,
  token_type: "Bearer"
}

// 2. Dùng access token lấy user info
GET https://api.tammi.vn/api/v1/user/user-info-by-scope
Authorization: Bearer {access_token}

// Response:
{
  username: "user123",
  phoneNumber: "84987654321"
}

// 3. Tạo hoặc cập nhật user trong database
// 4. Tạo session/token cho miniapp
// 5. Trả về miniapp token
```

## Mã lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `USER_DENIED` | User từ chối cấp quyền | Giải thích lý do cần quyền, yêu cầu user thử lại |
| `NO_PERMISSION` | Miniapp chưa được cấp quyền SSO | Apply quyền qua Dashboard |
| `INVALID_APP_ID` | AppId không đúng hoặc không tồn tại | Kiểm tra lại AppId |
| `NETWORK_ERROR` | Lỗi kết nối | Kiểm tra kết nối mạng |
| `WINDVANE_NOT_AVAILABLE` | Không chạy trong Tammi app | Chỉ test trong môi trường production |

## Best Practices

### 1. Kiểm tra môi trường trước khi gọi

```javascript
const isTammiApp = () => {
  return typeof window.WindVane !== 'undefined';
};

const login = async () => {
  if (!isTammiApp()) {
    console.warn('Not running in Tammi app - SSO unavailable');
    showManualLoginForm();
    return;
  }

  try {
    const authCode = await getAuthCode('YOUR_APP_ID');
    // ... xử lý authCode
  } catch (error) {
    console.error('SSO error:', error);
  }
};
```

### 2. Xử lý timeout

```javascript
const getAuthCodeWithTimeout = (appId, scopes, timeout = 10000) => {
  return Promise.race([
    getAuthCode(appId, scopes),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
};

// Sử dụng
try {
  const code = await getAuthCodeWithTimeout('YOUR_APP_ID', ['auth_user'], 5000);
} catch (error) {
  if (error.message === 'Timeout') {
    alert('Quá thời gian chờ. Vui lòng thử lại.');
  }
}
```

### 3. Retry logic

```javascript
const getAuthCodeWithRetry = async (appId, scopes, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await getAuthCode(appId, scopes);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
};
```

## Lưu ý bảo mật

:::danger KHÔNG lưu authCode
- **KHÔNG** lưu authCode vào localStorage, sessionStorage, hoặc cookie
- **KHÔNG** log authCode ra console trong production
- authCode chỉ nên tồn tại trong memory và gửi ngay lên backend
:::

:::warning One-time use
authCode chỉ sử dụng được **một lần**. Sau khi backend đổi lấy access token, authCode sẽ không còn giá trị.
:::

:::info Client ID & Secret
`CLIENT_ID` và `CLIENT_SECRET` để đổi authCode lấy access token chỉ được sử dụng ở **backend**. Không bao giờ để lộ ở frontend code.
:::

## Xem thêm

- [Cơ chế SSO](../../sso_auth/cach_thuc_hoat_dong) - Hiểu luồng SSO chi tiết
- [Triển khai Backend](../../sso_auth/backend) - Xử lý authCode ở backend
- [API getUserInfo](../../sso_auth/backend#api-2-get-user-info) - Lấy thông tin user
