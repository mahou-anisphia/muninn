---
sidebar_position: 4
---

# onLocationChange

Lắng nghe sự kiện thay đổi vị trí theo thời gian thực (real-time location tracking).

## Cú pháp

```javascript
// Start tracking
window.WindVane.call(
  'WVLocation',
  'onLocationChange',
  {
    enableHighAccuracy: 'true',
    distanceFilter: 10      // Khoảng cách tối thiểu để trigger (meters)
  },
  function(result) {
    // Callback được gọi mỗi khi vị trí thay đổi
    console.log('New location:', result.coords);
  },
  function(error) {
    console.error('Location error:', error);
  }
);

// Stop tracking
window.WindVane.call('WVLocation', 'offLocationChange', {});
```

## Tham số đầu vào

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|-------|
| `enableHighAccuracy` | `string` | `'true'` | `'true'` = GPS (chính xác), `'false'` = Network |
| `distanceFilter` | `number` | `10` | Khoảng cách tối thiểu (meters) để trigger callback |

## Success Callback

Callback được gọi **mỗi khi vị trí thay đổi** đủ `distanceFilter`:

```typescript
{
  coords: {
    latitude: number,        // Vĩ độ
    longitude: number,       // Kinh độ
    accuracy: number,        // Độ chính xác (meters)
    altitude: number,        // Độ cao (meters)
    heading: number,         // Hướng di chuyển (0-360 degrees)
    speed: number            // Tốc độ (m/s)
  },
  timestamp: number          // Thời điểm cập nhật (ms)
}
```

## Fail Callback

| Error Code | Mô tả |
|------------|-------|
| `PERMISSION_DENIED` | Chưa cấp quyền GPS |
| `POSITION_UNAVAILABLE` | GPS không khả dụng |
| `GPS_DISABLED` | GPS bị tắt trong Settings |

## Ví dụ

### 1. Real-time tracking cơ bản

```javascript
// Bắt đầu tracking
window.WindVane.call('WVLocation', 'onLocationChange', {
  enableHighAccuracy: 'true',
  distanceFilter: 10  // Update mỗi 10 meters
}, (location) => {
  console.log(`Vị trí mới: ${location.coords.latitude}, ${location.coords.longitude}`);
  updateMapMarker(location.coords);
}, (error) => {
  console.error('Tracking error:', error);
});

// Dừng tracking khi không cần
window.addEventListener('beforeunload', () => {
  window.WindVane.call('WVLocation', 'offLocationChange', {});
});
```

### 2. Delivery tracking

```javascript
class DeliveryTracker {
  constructor() {
    this.isTracking = false;
    this.path = [];
  }

  start() {
    if (this.isTracking) return;
    
    this.isTracking = true;
    window.WindVane.call('WVLocation', 'onLocationChange', {
      enableHighAccuracy: 'true',
      distanceFilter: 20  // Update mỗi 20m
    }, (location) => {
      this.onLocationUpdate(location);
    }, (error) => {
      console.error('Tracking error:', error);
      this.stop();
    });
  }

  onLocationUpdate(location) {
    const point = {
      lat: location.coords.latitude,
      lng: location.coords.longitude,
      timestamp: location.timestamp,
      speed: location.coords.speed,
      accuracy: location.coords.accuracy
    };
    
    this.path.push(point);
    
    // Update server
    this.sendLocationToServer(point);
    
    // Update UI
    this.updateMap(point);
    
    // Tính toán ETA
    this.calculateETA();
  }

  async sendLocationToServer(point) {
    try {
      await fetch('/api/delivery/location', {
        method: 'POST',
        body: JSON.stringify(point)
      });
    } catch (error) {
      console.error('Failed to send location:', error);
    }
  }

  updateMap(point) {
    // Vẽ path trên map
    drawPolyline(this.path);
    
    // Di chuyển marker
    moveMarker(point.lat, point.lng);
  }

  calculateETA() {
    // Tính ETA dựa trên distance và speed
    const totalDistance = calculatePathDistance(this.path);
    const avgSpeed = calculateAverageSpeed(this.path);
    const eta = totalDistance / avgSpeed;
    
    updateETADisplay(eta);
  }

  stop() {
    if (!this.isTracking) return;
    
    this.isTracking = false;
    window.WindVane.call('WVLocation', 'offLocationChange', {});
    
    // Lưu tracking session
    this.saveSession();
  }

  saveSession() {
    const session = {
      path: this.path,
      distance: calculatePathDistance(this.path),
      duration: this.path[this.path.length - 1].timestamp - this.path[0].timestamp,
      avgSpeed: calculateAverageSpeed(this.path)
    };
    
    localStorage.setItem('lastSession', JSON.stringify(session));
  }
}

// Sử dụng
const tracker = new DeliveryTracker();
tracker.start();

// Dừng khi hoàn thành delivery
document.getElementById('complete-btn').addEventListener('click', () => {
  tracker.stop();
});
```

### 3. Fitness tracking app

```javascript
class FitnessTracker {
  constructor() {
    this.isRunning = false;
    this.startTime = null;
    this.distance = 0;
    this.lastPosition = null;
    this.checkpoints = [];
  }

  startWorkout() {
    this.isRunning = true;
    this.startTime = Date.now();
    
    window.WindVane.call('WVLocation', 'onLocationChange', {
      enableHighAccuracy: 'true',
      distanceFilter: 5  // Update mỗi 5m (chính xác hơn)
    }, (location) => {
      this.onLocationUpdate(location);
    });
    
    // Update UI mỗi giây
    this.uiInterval = setInterval(() => {
      this.updateUI();
    }, 1000);
  }

  onLocationUpdate(location) {
    const currentPos = {
      lat: location.coords.latitude,
      lng: location.coords.longitude,
      timestamp: location.timestamp
    };
    
    // Tính distance
    if (this.lastPosition) {
      const dist = calculateDistance(
        this.lastPosition.lat,
        this.lastPosition.lng,
        currentPos.lat,
        currentPos.lng
      );
      this.distance += dist;
    }
    
    this.lastPosition = currentPos;
    this.checkpoints.push(currentPos);
    
    // Update stats
    this.updateStats(location.coords);
  }

  updateStats(coords) {
    const duration = Date.now() - this.startTime;
    const durationMin = Math.floor(duration / 60000);
    const pace = durationMin / (this.distance / 1000); // min/km
    const speed = coords.speed * 3.6; // m/s -> km/h
    
    // Broadcast stats
    this.broadcastStats({
      distance: this.distance,
      duration: durationMin,
      pace: pace,
      speed: speed,
      calories: this.calculateCalories()
    });
  }

  calculateCalories() {
    // Công thức đơn giản: 65 cal/km (tùy cân nặng)
    const distanceKm = this.distance / 1000;
    return Math.floor(distanceKm * 65);
  }

  updateUI() {
    document.getElementById('distance').textContent = 
      (this.distance / 1000).toFixed(2) + ' km';
    
    const duration = Date.now() - this.startTime;
    const minutes = Math.floor(duration / 60000);
    const seconds = Math.floor((duration % 60000) / 1000);
    document.getElementById('time').textContent = 
      `${minutes}:${seconds.toString().padStart(2, '0')}`;
    
    document.getElementById('calories').textContent = 
      this.calculateCalories() + ' cal';
  }

  stopWorkout() {
    this.isRunning = false;
    
    window.WindVane.call('WVLocation', 'offLocationChange', {});
    clearInterval(this.uiInterval);
    
    // Save workout
    this.saveWorkout();
  }

  saveWorkout() {
    const workout = {
      date: new Date().toISOString(),
      distance: this.distance,
      duration: Date.now() - this.startTime,
      calories: this.calculateCalories(),
      checkpoints: this.checkpoints
    };
    
    // Save to backend
    fetch('/api/workouts', {
      method: 'POST',
      body: JSON.stringify(workout)
    });
  }
}
```

### 4. React Hook

```jsx
import { useState, useEffect, useRef } from 'react';

function useLocationTracking(options = {}) {
  const [location, setLocation] = useState(null);
  const [isTracking, setIsTracking] = useState(false);
  const [error, setError] = useState(null);
  const pathRef = useRef([]);

  const startTracking = () => {
    setIsTracking(true);
    setError(null);
    
    window.WindVane.call('WVLocation', 'onLocationChange',
      {
        enableHighAccuracy: options.highAccuracy ? 'true' : 'false',
        distanceFilter: options.distanceFilter || 10
      },
      (result) => {
        setLocation(result);
        pathRef.current.push({
          lat: result.coords.latitude,
          lng: result.coords.longitude,
          timestamp: result.timestamp
        });
        
        // Callback
        if (options.onLocationChange) {
          options.onLocationChange(result, pathRef.current);
        }
      },
      (err) => {
        setError(err);
        setIsTracking(false);
      }
    );
  };

  const stopTracking = () => {
    setIsTracking(false);
    window.WindVane.call('WVLocation', 'offLocationChange', {});
  };

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (isTracking) {
        window.WindVane.call('WVLocation', 'offLocationChange', {});
      }
    };
  }, [isTracking]);

  return {
    location,
    isTracking,
    error,
    path: pathRef.current,
    startTracking,
    stopTracking
  };
}

// Component sử dụng
function RunTracker() {
  const { location, isTracking, path, startTracking, stopTracking } = 
    useLocationTracking({
      highAccuracy: true,
      distanceFilter: 5,
      onLocationChange: (loc, fullPath) => {
        console.log('Location updated:', loc);
      }
    });

  const totalDistance = useMemo(() => {
    return calculateTotalDistance(path);
  }, [path]);

  return (
    <div>
      <button onClick={isTracking ? stopTracking : startTracking}>
        {isTracking ? 'Dừng' : 'Bắt đầu'}
      </button>
      
      {location && (
        <div>
          <p>Vị trí: {location.coords.latitude}, {location.coords.longitude}</p>
          <p>Tốc độ: {(location.coords.speed * 3.6).toFixed(1)} km/h</p>
          <p>Khoảng cách: {(totalDistance / 1000).toFixed(2)} km</p>
          <p>Số điểm: {path.length}</p>
        </div>
      )}
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Stop khi không cần**: Luôn gọi `offLocationChange` khi không tracking
- **Batch upload**: Gửi nhiều location cùng lúc thay vì từng cái
- **Battery aware**: Tăng `distanceFilter` khi pin yếu
- **Validate accuracy**: Bỏ qua location có `accuracy` quá lớn (> 100m)
- **Background handling**: Dừng tracking khi app vào background

### ❌ Không nên

- Quên stop tracking (tốn pin)
- Upload location mỗi lần update (tốn network)
- Dùng `distanceFilter` quá nhỏ (< 5m, tốn pin)
- Track trong thời gian dài mà không cần thiết

## Use Cases

| Use Case | distanceFilter | highAccuracy | Xử lý |
|----------|----------------|--------------|-------|
| **Delivery tracking** | 20-50m | `true` | Batch upload mỗi 30s |
| **Fitness tracking** | 5-10m | `true` | Local save, sync sau |
| **Navigation** | 10m | `true` | Real-time update |
| **Geofencing** | 50-100m | `false` | Check khi vào/ra zone |

## Battery Optimization

```javascript
// Tăng distanceFilter khi pin yếu
async function startSmartTracking() {
  const battery = await getBatteryInfo();
  
  const distanceFilter = battery.level < 20 ? 50 : 
                        battery.level < 50 ? 20 : 10;
  
  window.WindVane.call('WVLocation', 'onLocationChange', {
    enableHighAccuracy: battery.level > 20 ? 'true' : 'false',
    distanceFilter: distanceFilter
  }, handleLocationUpdate);
}
```

## Security & Privacy

:::danger Background Location
API này chỉ hoạt động khi miniapp ở **foreground**. Không thể tracking khi:
- User thoát app
- App chuyển sang background
- Màn hình tắt

Nếu cần background tracking, contact Viettel để được hỗ trợ.
:::

## Giới hạn

| Giới hạn | Giá trị |
|----------|---------|
| Tracking duration | Không giới hạn (trong foreground) |
| distanceFilter tối thiểu | 1m (khuyến nghị ≥ 5m) |
| Update frequency | Tùy GPS (thường 1-5s) |

## Xem thêm

- [getLocation](./getLocation) - Lấy vị trí một lần
- [getBatteryInfo](../C_device_hardware/getBatteryInfo) - Thông tin pin
