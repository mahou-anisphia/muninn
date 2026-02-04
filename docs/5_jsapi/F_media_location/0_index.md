---
sidebar_position: 0
id: index
---

# Media & Location APIs

Các API để xử lý media (ảnh, video) và định vị địa lý.

## Danh sách APIs

### Image
- **takePhoto** - Chụp ảnh hoặc chọn từ thư viện
- **compressImage** - Nén ảnh để giảm dung lượng
- **previewImage** - Xem ảnh toàn màn hình

### Video
- **chooseVideo** - Chọn video từ thư viện
- **recordVideo** - Quay video

### Location
- **getLocation** - Lấy tọa độ hiện tại
- **onLocationChange** - Theo dõi thay đổi vị trí
- **openMap** - Mở bản đồ tại tọa độ

## Use Cases phổ biến

### 1. Chụp và upload avatar

```javascript
const updateAvatar = async () => {
  window.WindVane.call('WVCamera', 'takePhoto', 
    { 
      mode: 'both',      // Camera hoặc thư viện
      sourceType: 'camera,album'
    },
    async (result) => {
      // Nén ảnh trước khi upload
      window.WindVane.call('WVImage', 'compressImage', 
        { 
          src: result.url,
          quality: 80  // 0-100
        },
        async (compressed) => {
          // Upload lên server
          await uploadAvatar(compressed.path);
          showToast('Đã cập nhật avatar');
        }
      );
    }
  );
};
```

### 2. Định vị và hiển thị trên bản đồ

```javascript
const showCurrentLocation = () => {
  window.WindVane.call('WVLocation', 'getLocation', 
    { 
      enableHighAccuracy: 'true',
      timeout: 10000
    },
    (result) => {
      const { latitude, longitude } = result.coords;
      
      // Mở bản đồ tại vị trí hiện tại
      window.WindVane.call('WVMap', 'openMap', {
        latitude,
        longitude,
        name: 'Vị trí hiện tại',
        address: result.address
      });
    }
  );
};
```

### 3. Gallery với preview

```javascript
const ImageGallery = ({ images }) => {
  const handleImageClick = (index) => {
    window.WindVane.call('WVImage', 'previewImage', {
      urls: images.map(img => img.url),
      current: images[index].url
    });
  };

  return (
    <div className="gallery">
      {images.map((img, i) => (
        <img 
          key={i} 
          src={img.url} 
          onClick={() => handleImageClick(i)}
        />
      ))}
    </div>
  );
};
```

### 4. Tracking vị trí real-time

```javascript
const startLocationTracking = () => {
  window.WindVane.call('WVLocation', 'onLocationChange', 
    { enableHighAccuracy: 'true' },
    (location) => {
      // Callback được gọi mỗi khi vị trí thay đổi
      updateMapMarker(location.coords.latitude, location.coords.longitude);
      
      // Gửi lên server
      sendLocationUpdate(location.coords);
    }
  );
};

const stopLocationTracking = () => {
  window.WindVane.call('WVLocation', 'offLocationChange', {});
};
```

### 5. Chọn địa điểm từ bản đồ

```javascript
const chooseLocation = () => {
  return new Promise((resolve) => {
    window.WindVane.call('WVMap', 'chooseLocation', {}, (result) => {
      resolve({
        name: result.name,
        address: result.address,
        latitude: result.latitude,
        longitude: result.longitude
      });
    });
  });
};

// Sử dụng
const location = await chooseLocation();
console.log('User chọn:', location.address);
```

## Quyền hạn

### Location APIs
- **Yêu cầu**: User consent khi gọi lần đầu
- **Lưu ý**: Chỉ yêu cầu khi cần thiết, giải thích rõ lý do

### Camera/Media APIs
- **Yêu cầu**: Phê duyệt từ Dashboard + user consent
- **Lưu ý**: Không lưu ảnh/video người dùng mà không cho phép

## Best Practices

### 1. Compress ảnh trước khi upload

```javascript
const uploadPhoto = async (photoPath) => {
  // Bước 1: Nén ảnh
  const compressed = await windvaneCall('WVImage', 'compressImage', {
    src: photoPath,
    quality: 70  // Cân bằng giữa chất lượng và kích thước
  });

  // Bước 2: Upload
  await windvaneCall('WVFile', 'uploadFile', {
    url: 'https://api.example.com/upload',
    filePath: compressed.path
  });
};
```

### 2. Xử lý lỗi GPS

```javascript
const getLocationSafe = async () => {
  try {
    const result = await windvaneCall('WVLocation', 'getLocation', {
      enableHighAccuracy: 'true',
      timeout: 5000
    });
    return result.coords;
  } catch (error) {
    if (error.error === 'TIMEOUT') {
      // Thử lại với độ chính xác thấp hơn
      return await windvaneCall('WVLocation', 'getLocation', {
        enableHighAccuracy: 'false'
      });
    } else if (error.error === 'PERMISSION_DENIED') {
      alert('Vui lòng cấp quyền truy cập vị trí');
      throw error;
    }
  }
};
```

### 3. Cache location

```javascript
let cachedLocation = null;
let locationExpiry = null;

const getCurrentLocation = async (maxAge = 60000) => { // Cache 1 phút
  if (cachedLocation && locationExpiry > Date.now()) {
    return cachedLocation;
  }

  const result = await windvaneCall('WVLocation', 'getLocation', {});
  cachedLocation = result.coords;
  locationExpiry = Date.now() + maxAge;
  
  return cachedLocation;
};
```

## Lưu ý bảo mật & quyền riêng tư

:::warning Không theo dõi vị trí background
Miniapp không thể theo dõi vị trí khi chạy background. Chỉ lấy vị trí khi app đang active.
:::

:::danger Xin quyền có ngữ cảnh
Luôn giải thích tại sao cần quyền **trước** khi yêu cầu:

```javascript
// ✓ Tốt
showDialog({
  title: 'Yêu cầu quyền vị trí',
  message: 'Miniapp cần biết vị trí để tìm cửa hàng gần bạn',
  onConfirm: () => getLocation()
});

// ✗ Tránh - không giải thích
getLocation(); // User không biết tại sao cần quyền
```
:::

## Xem thêm

- [Device Hardware](../C_device_hardware/index) - APIs phần cứng khác
- [Sample Code](https://github.com/mahou-anisphia/miniapp-sample-code) - Ví dụ thực tế
