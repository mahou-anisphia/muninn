---
sidebar_position: 3
---

# getLocation

Lấy tọa độ vị trí hiện tại của thiết bị (GPS/Network).

## Cú pháp

```javascript
window.WindVane.call(
  'WVLocation',
  'getLocation',
  {
    enableHighAccuracy: 'true' | 'false',
    timeout: number,
    maximumAge: number
  },
  (result) => { /* success */ },
  (error) => { /* fail */ }
);
```

## Tham số đầu vào

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `enableHighAccuracy` | `string` | Không | `'true'` = GPS (chính xác cao), `'false'` = Network (nhanh hơn) - Mặc định: `'true'` |
| `timeout` | `number` | Không | Timeout (ms) - Mặc định: 10000 (10s) |
| `maximumAge` | `number` | Không | Cache time (ms) - Mặc định: 60000 (60s) |

## Kết quả trả về

**Success callback:**

```javascript
{
  coords: {
    latitude: number,        // Vĩ độ
    longitude: number,       // Kinh độ
    accuracy: number,        // Độ chính xác (meters)
    altitude: number,        // Độ cao (meters)
    altitudeAccuracy: number,// Độ chính xác độ cao
    heading: number,         // Hướng di chuyển (degrees)
    speed: number            // Tốc độ (m/s)
  },
  timestamp: number          // Thời điểm lấy vị trí (ms)
}
```

**Fail callback:**

```javascript
{
  error: string,             // Mã lỗi
  errorMessage: string       // Mô tả lỗi
}
```

## Ví dụ

### 1. Lấy vị trí cơ bản

```javascript
window.WindVane.call('WVLocation', 'getLocation',
  { enableHighAccuracy: 'true' },
  (result) => {
    const { latitude, longitude } = result.coords;
    console.log(`Vị trí: ${latitude}, ${longitude}`);
    console.log(`Độ chính xác: ${result.coords.accuracy}m`);
  },
  (error) => {
    console.error('Lỗi GPS:', error.errorMessage);
  }
);
```

### 2. Lấy vị trí nhanh với Network

```javascript
// Sử dụng Network location (nhanh hơn GPS)
window.WindVane.call('WVLocation', 'getLocation',
  {
    enableHighAccuracy: 'false',
    timeout: 5000
  },
  (result) => {
    displayOnMap(result.coords.latitude, result.coords.longitude);
  },
  (error) => {
    console.error('Không lấy được vị trí:', error);
  }
);
```

### 3. Promise wrapper với cache

```javascript
let cachedLocation = null;
let cacheTime = null;

const getLocation = (useCache = true, highAccuracy = true) => {
  // Kiểm tra cache (60s)
  if (useCache && cachedLocation && Date.now() - cacheTime < 60000) {
    return Promise.resolve(cachedLocation);
  }

  return new Promise((resolve, reject) => {
    window.WindVane.call('WVLocation', 'getLocation',
      {
        enableHighAccuracy: highAccuracy ? 'true' : 'false',
        timeout: 10000
      },
      (result) => {
        cachedLocation = result;
        cacheTime = Date.now();
        resolve(result);
      },
      reject
    );
  });
};

// Sử dụng
async function showMyLocation() {
  try {
    const location = await getLocation();
    console.log('Vị trí:', location.coords);
  } catch (error) {
    console.error('Lỗi:', error);
  }
}
```

### 4. React Hook với retry logic

```jsx
import { useState, useEffect } from 'react';

function useLocation(autoFetch = false) {
  const [location, setLocation] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchLocation = async (retries = 3) => {
    setLoading(true);
    setError(null);

    for (let i = 0; i < retries; i++) {
      try {
        const result = await new Promise((resolve, reject) => {
          window.WindVane.call('WVLocation', 'getLocation',
            {
              enableHighAccuracy: 'true',
              timeout: 10000
            },
            resolve,
            reject
          );
        });
        
        setLocation(result);
        setLoading(false);
        return result;
      } catch (err) {
        if (i === retries - 1) {
          setError(err);
          setLoading(false);
          throw err;
        }
        // Đợi 2s trước khi retry
        await new Promise(r => setTimeout(r, 2000));
      }
    }
  };

  useEffect(() => {
    if (autoFetch) {
      fetchLocation();
    }
  }, [autoFetch]);

  return { location, loading, error, fetchLocation };
}

// Component sử dụng
function LocationDisplay() {
  const { location, loading, error, fetchLocation } = useLocation(true);

  if (loading) return <div>Đang lấy vị trí...</div>;
  if (error) return <div>Lỗi: {error.errorMessage}</div>;
  if (!location) return <button onClick={fetchLocation}>Lấy vị trí</button>;

  return (
    <div>
      <p>Vĩ độ: {location.coords.latitude}</p>
      <p>Kinh độ: {location.coords.longitude}</p>
      <p>Độ chính xác: {location.coords.accuracy}m</p>
    </div>
  );
}
```

### 5. Tích hợp với map

```javascript
const showLocationOnMap = async () => {
  try {
    const location = await getLocation();
    
    // Khởi tạo map (ví dụ với Google Maps)
    const map = new google.maps.Map(document.getElementById('map'), {
      center: {
        lat: location.coords.latitude,
        lng: location.coords.longitude
      },
      zoom: 15
    });
    
    // Thêm marker
    new google.maps.Marker({
      position: {
        lat: location.coords.latitude,
        lng: location.coords.longitude
      },
      map: map,
      title: 'Vị trí của bạn'
    });
    
    // Hiển thị accuracy circle
    new google.maps.Circle({
      strokeColor: '#4285F4',
      strokeOpacity: 0.8,
      strokeWeight: 2,
      fillColor: '#4285F4',
      fillOpacity: 0.15,
      map: map,
      center: {
        lat: location.coords.latitude,
        lng: location.coords.longitude
      },
      radius: location.coords.accuracy
    });
    
  } catch (error) {
    console.error('Không thể hiển thị map:', error);
  }
};
```

## Use Cases

| Use Case | Cấu hình khuyến nghị |
|----------|----------------------|
| **Check-in tại địa điểm** | `enableHighAccuracy: 'true'` |
| **Tìm cửa hàng gần nhất** | `enableHighAccuracy: 'false'` (nhanh hơn) |
| **Tracking di chuyển** | Dùng `onLocationChange` thay vì `getLocation` |
| **Hiển thị thành phố hiện tại** | `enableHighAccuracy: 'false', maximumAge: 300000` (5 phút) |

## Best Practices

### 1. Cache vị trí

```javascript
// Tránh gọi API quá nhiều lần
const LocationCache = {
  data: null,
  timestamp: null,
  maxAge: 60000, // 60 giây

  async get(forceRefresh = false) {
    if (!forceRefresh && this.data && 
        Date.now() - this.timestamp < this.maxAge) {
      return this.data;
    }

    const location = await getLocation();
    this.data = location;
    this.timestamp = Date.now();
    return location;
  },

  clear() {
    this.data = null;
    this.timestamp = null;
  }
};
```

### 2. Fallback strategy

```javascript
const getLocationWithFallback = async () => {
  try {
    // Thử GPS trước (chính xác cao)
    return await getLocation(false, true);
  } catch (error) {
    console.warn('GPS failed, trying network location');
    try {
      // Fallback sang Network location
      return await getLocation(false, false);
    } catch (error2) {
      // Fallback sang IP geolocation
      return await getLocationFromIP();
    }
  }
};

const getLocationFromIP = async () => {
  const response = await fetch('https://ipapi.co/json/');
  const data = await response.json();
  return {
    coords: {
      latitude: data.latitude,
      longitude: data.longitude,
      accuracy: 50000 // IP location không chính xác
    }
  };
};
```

### 3. Permission check

```javascript
const checkLocationPermission = async () => {
  try {
    const result = await new Promise((resolve) => {
      window.WindVane.call('WVBase', 'canIUse',
        { api: 'getLocation' },
        resolve,
        () => resolve({ result: false })
      );
    });
    
    if (!result.result) {
      showToast('Cần cấp quyền truy cập vị trí');
      return false;
    }
    return true;
  } catch {
    return false;
  }
};
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Xử lý |
|--------|-------------|-------|
| `PERMISSION_DENIED` | User chưa cấp quyền GPS | Yêu cầu vào Settings |
| `POSITION_UNAVAILABLE` | GPS không khả dụng | Thử Network location |
| `TIMEOUT` | Quá thời gian chờ | Tăng timeout hoặc retry |
| `GPS_DISABLED` | GPS bị tắt trong Settings | Yêu cầu bật GPS |

## Performance Tips

:::tip Tối ưu performance
1. **Sử dụng cache**: Không gọi `getLocation` liên tục, cache 30-60s
2. **Network location cho UX**: Dùng `enableHighAccuracy: 'false'` khi cần phản hồi nhanh
3. **Background tracking**: Dùng `onLocationChange` thay vì polling `getLocation`
4. **Timeout hợp lý**: 5-10s là đủ, không để quá 30s
5. **Kiểm tra accuracy**: Nếu `accuracy > 100m`, có thể retry với GPS
:::

## Quyền yêu cầu

- **Location**: Cần approve trong Dashboard
- **User consent**: Hiển thị dialog xin quyền lần đầu
- **Background location**: Không hỗ trợ (chỉ khi app active)

:::warning Lưu ý
Location tracking chỉ hoạt động khi miniapp đang chạy (foreground). Không thể lấy vị trí khi app ở background.
:::

## Xem thêm

- [onLocationChange](./onLocationChange) - Lắng nghe thay đổi vị trí
- [canIUse](../A_app_base/canIUse) - Kiểm tra quyền truy cập
