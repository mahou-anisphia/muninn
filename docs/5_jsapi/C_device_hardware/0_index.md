---
sidebar_position: 0
id: index
---

# Device & Hardware APIs

Các API để truy cập phần cứng và cảm biến của thiết bị.

## Danh sách APIs

### Camera & Media
- **takePhoto** - Chụp ảnh hoặc chọn từ thư viện
- **chooseVideo** - Chọn video từ thư viện

### Location
- **getLocation** - Lấy tọa độ GPS hiện tại
- **onLocationChange** - Lắng nghe thay đổi vị trí

### Sensors
- **Accelerometer** - Cảm biến gia tốc
- **Compass** - La bàn điện tử
- **Gyroscope** - Con quay hồi chuyển

### Bluetooth
- **scan** - Quét thiết bị BLE
- **connect** - Kết nối thiết bị
- **getServices** - Lấy danh sách services
- **readValue** / **writeValue** - Đọc/ghi characteristic

### Other Hardware
- **scan** (QR/Barcode) - Quét mã QR/Barcode
- **getBatteryInfo** - Thông tin pin
- **getNetworkType** - Loại mạng đang kết nối
- **makePhoneCall** - Thực hiện cuộc gọi

## Use Cases phổ biến

### 1. Chụp ảnh và upload

```javascript
const takeAndUpload = async () => {
  // Bước 1: Chụp ảnh
  window.WindVane.call('WVCamera', 'takePhoto', 
    { mode: 'camera' },  // Chỉ dùng camera, không cho chọn từ thư viện
    async (result) => {
      // Bước 2: Upload lên server
      const formData = new FormData();
      formData.append('image', result.url);
      
      await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      showToast('Đã upload ảnh');
    }
  );
};
```

### 2. Lấy vị trí và hiển thị bản đồ

```javascript
const showUserLocation = () => {
  window.WindVane.call('WVLocation', 'getLocation', 
    { enableHighAccuracy: 'true' },
    (result) => {
      const { latitude, longitude } = result.coords;
      
      // Hiển thị trên bản đồ
      initMap(latitude, longitude);
      
      // Hoặc gửi lên server
      updateUserLocation(latitude, longitude);
    }
  );
};
```

### 3. Quét mã QR thanh toán

```javascript
const scanQRPayment = () => {
  window.WindVane.call('WVScan', 'scan', {}, (result) => {
    const qrContent = result.content;
    
    // Parse QR code
    if (qrContent.startsWith('VIETTEL_PAY:')) {
      processPayment(qrContent);
    } else {
      alert('Mã QR không hợp lệ');
    }
  });
};
```

### 4. Kết nối Bluetooth device (IoT)

```javascript
const connectBLEDevice = async (deviceId) => {
  try {
    // Kết nối
    await windvaneCall('WVBluetooth', 'connect', { deviceId });
    
    // Lấy services
    const services = await windvaneCall('WVBluetooth', 'getServices', { deviceId });
    
    // Đọc dữ liệu từ characteristic
    const data = await windvaneCall('WVBluetooth', 'readValue', {
      deviceId,
      serviceId: services[0].uuid,
      characteristicId: 'xxxx-xxxx-xxxx'
    });
    
    console.log('Device data:', data);
  } catch (error) {
    console.error('BLE error:', error);
  }
};

// Helper function
const windvaneCall = (module, method, params) => {
  return new Promise((resolve, reject) => {
    window.WindVane.call(module, method, params, resolve, reject);
  });
};
```

## Quyền hạn yêu cầu

Hầu hết các API phần cứng yêu cầu:
1. **Phê duyệt từ Viettel** qua Dashboard
2. **User consent** - User phải cấp quyền khi sử dụng lần đầu

:::warning Kiểm tra quyền trước khi dùng
Luôn dùng `canIUse` để kiểm tra quyền trước khi gọi API phần cứng:

```javascript
window.WindVane.call('WVBase', 'canIUse', { api: 'takePhoto' }, (result) => {
  if (result.result) {
    // Có quyền - tiếp tục
  } else {
    alert('Miniapp chưa được cấp quyền Camera');
  }
});
```
:::

## Lưu ý bảo mật

- **Location**: Chỉ yêu cầu khi thực sự cần thiết cho nghiệp vụ
- **Camera**: Giải thích rõ lý do cần quyền camera
- **Bluetooth**: Chỉ kết nối thiết bị đã được user chọn
- **Battery**: Thông tin pin không nhạy cảm nhưng không nên lạm dụng

## Best Practices

### 1. Graceful fallback

```javascript
const getLocationWithFallback = async () => {
  try {
    const result = await windvaneCall('WVLocation', 'getLocation', {});
    return result.coords;
  } catch (error) {
    // Fallback: yêu cầu user nhập địa chỉ thủ công
    return await showAddressInput();
  }
};
```

### 2. Request quyền với context rõ ràng

```javascript
// ✓ Tốt - giải thích trước khi yêu cầu
const requestCamera = () => {
  showDialog({
    title: 'Yêu cầu quyền Camera',
    message: 'Miniapp cần Camera để bạn có thể chụp ảnh sản phẩm',
    onConfirm: () => {
      window.WindVane.call('WVCamera', 'takePhoto', {}, ...);
    }
  });
};
```

### 3. Xử lý timeout

```javascript
const getLocationWithTimeout = (timeout = 10000) => {
  return Promise.race([
    windvaneCall('WVLocation', 'getLocation', {}),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
};
```

## Xem thêm

- [Quyền hạn](../1_truoc_khi_bat_dau) - Hiểu về permissions
- [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) - Ví dụ thực tế
