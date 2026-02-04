---
sidebar_position: 1
---

# Trước khi bắt đầu

jsAPI cho phép truy cập tính năng native, nhưng cũng yêu cầu **phê duyệt quyền** từ Superapp để đảm bảo bảo mật.

## Quyền hạn (Permissions)

Khác với các nền tảng khác, Tammi Superapp không phân chia cứng nhắc giữa "Ordinary" và "Special" permission cho developer. Thay vào đó:

- **Mặc định**: Hầu hết API đều bị tắt (disabled) hoặc cần whitelist
- **Cấp quyền**: Quyền được cấp theo từng Miniapp ID (AppId) dựa trên cấu hình từ phía quản trị viên hệ thống (Viettel)

:::tip Tự kiểm tra quyền (Self-Check)
Do cấu hình quyền có thể thay đổi, Miniapp được khuyến nghị **luôn kiểm tra tính khả dụng** của API trước khi gọi, để tránh lỗi runtime.
:::

## Kiểm tra quyền và API khả dụng

### Cách 1: Sử dụng `canIUse` (Khuyến nghị)

Sử dụng API `canIUse` để kiểm tra xem một phương thức (method) hoặc class cụ thể có được hỗ trợ và cho phép hay không.

```javascript
window.WindVane.call('WVBase', 'canIUse', {
  api: 'startNotifications' // Tên API cần check
}, (result) => {
  if (result.result) {
    console.log('API được phép sử dụng');
  } else {
    console.log('API không khả dụng hoặc chưa được cấp quyền');
  }
});
```

### Cách 2: Try-Catch và xử lý lỗi

Luôn bao bọc lời gọi API trong block xử lý lỗi để nhận diện trường hợp bị chặn quyền.

```javascript
window.WindVane.call('WVCamera', 'takePhoto', {},
  (success) => { 
    console.log('Ảnh đã chụp:', success.url);
  },
  (error) => {
    if (error.error === 'NO_PERMISSION' || error.errorCode === '403') {
       alert('Bạn chưa được cấp quyền sử dụng Camera');
    }
  }
);
```

### Cách 3: Kiểm tra qua Dashboard

1. Đăng nhập [Dashboard](https://dashboard.tammi.vn)
2. Chọn miniapp của bạn
3. Vào tab **JSAPI Feature**
4. Xem danh sách API đã được kích hoạt (Active) cho AppId của bạn

## Xin cấp quyền mới

Nếu cần sử dụng API chưa có trong danh sách cho phép:

**Bước 1:** Vào Dashboard > Miniapp > **JSAPI Feature**

**Bước 2:** Nhấn **Apply** và chọn API cần sử dụng

**Bước 3:** Điền lý do nghiệp vụ (Business Case) rõ ràng

**Bước 4:** Submit và chờ phê duyệt

:::info Thời gian phê duyệt
Thông thường mất **1-2 ngày làm việc** để Viettel xem xét và phê duyệt quyền sử dụng API.
:::

## Xử lý lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `NO_PERMISSION` / `403` | Miniapp chưa được cấp quyền | Xin cấp quyền qua Dashboard |
| `API_NOT_FOUND` | API không tồn tại hoặc tên sai | Kiểm tra lại tên module/method |
| `INVALID_PARAMS` | Tham số không hợp lệ | Xem lại tài liệu API |
| `USER_DENIED` | User từ chối cấp quyền | Giải thích rõ lý do cần quyền |

## Best Practices

### 1. Luôn kiểm tra quyền trước khi gọi

```javascript
const checkAndUseCamera = async () => {
  // Kiểm tra quyền
  window.WindVane.call('WVBase', 'canIUse', { api: 'takePhoto' }, (result) => {
    if (result.result) {
      // Có quyền - gọi API
      window.WindVane.call('WVCamera', 'takePhoto', {}, successCallback, errorCallback);
    } else {
      // Không có quyền - hiển thị thông báo
      alert('Miniapp chưa được cấp quyền sử dụng Camera');
    }
  });
};
```

### 2. Xử lý fallback khi API không khả dụng

```javascript
const getLocation = () => {
  window.WindVane.call('WVLocation', 'getLocation', {},
    (result) => {
      console.log('Vị trí:', result.coords);
    },
    (error) => {
      // Fallback: yêu cầu user nhập địa chỉ thủ công
      showManualAddressInput();
    }
  );
};
```

### 3. Cung cấp thông tin rõ ràng về quyền

Khi yêu cầu quyền, giải thích tại sao miniapp cần quyền đó:

```javascript
const requestCameraPermission = () => {
  showDialog({
    title: 'Cấp quyền Camera',
    message: 'Miniapp cần quyền truy cập Camera để bạn có thể quét mã QR thanh toán',
    onConfirm: () => {
      window.WindVane.call('WVCamera', 'takePhoto', {}, ...);
    }
  });
};
```

## Tiếp theo

Sau khi hiểu về quyền hạn, bạn có thể:

- Xem [Danh mục jsAPI](./A_app_base/index) để tìm API phù hợp
- Tham khảo [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) để xem ví dụ thực tế
