---
sidebar_position: 11
---

# Compass

Truy cập la bàn điện tử để xác định hướng thiết bị.

## Cú pháp

```javascript
// Bắt đầu lắng nghe
window.WindVane.call('WVMotion', 'startCompass', params, successCallback, failCallback);

// Dừng lắng nghe
window.WindVane.call('WVMotion', 'stopCompass', {});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `interval` | `string` | Không | `'ui'` (mặc định): 30Hz<br/>`'game'`: 60Hz<br/>`'normal'`: 10Hz |

## Success Callback

```javascript
{
  direction: number,     // Hướng từ Bắc (0-360 degrees)
  accuracy: number,      // Độ chính xác (degrees)
  timestamp: number      // Thời điểm đo (ms)
}
```

### Quy ước hướng

```
           0° (Bắc)
              |
              |
270° (Tây) ---+--- 90° (Đông)
              |
              |
          180° (Nam)
```

## Ví dụ

### 1. Hiển thị hướng cơ bản

```javascript
window.WindVane.call('WVMotion', 'startCompass', 
  { interval: 'ui' },
  (result) => {
    const direction = result.direction;
    const compass = getCompassDirection(direction);
    
    document.getElementById('direction').textContent = 
      `${Math.round(direction)}° ${compass}`;
  },
  (error) => {
    console.error('Compass error:', error);
  }
);

function getCompassDirection(degrees) {
  const directions = ['Bắc', 'Đông Bắc', 'Đông', 'Đông Nam', 
                     'Nam', 'Tây Nam', 'Tây', 'Tây Bắc'];
  const index = Math.round(degrees / 45) % 8;
  return directions[index];
}
```

### 2. Compass UI với kim chỉ

```html
<style>
  .compass {
    width: 200px;
    height: 200px;
    border: 2px solid #333;
    border-radius: 50%;
    position: relative;
    background: #f0f0f0;
  }
  
  .needle {
    width: 4px;
    height: 80px;
    background: red;
    position: absolute;
    left: 50%;
    top: 20px;
    transform-origin: center 80px;
    transition: transform 0.3s ease;
  }
  
  .label {
    position: absolute;
    font-weight: bold;
  }
  
  .label.n { top: 10px; left: 50%; transform: translateX(-50%); }
  .label.s { bottom: 10px; left: 50%; transform: translateX(-50%); }
  .label.e { right: 10px; top: 50%; transform: translateY(-50%); }
  .label.w { left: 10px; top: 50%; transform: translateY(-50%); }
</style>

<div class="compass">
  <div class="label n">N</div>
  <div class="label s">S</div>
  <div class="label e">E</div>
  <div class="label w">W</div>
  <div class="needle" id="needle"></div>
</div>
<div id="info"></div>
```

```javascript
class CompassUI {
  constructor() {
    this.needle = document.getElementById('needle');
    this.info = document.getElementById('info');
    this.isActive = false;
  }

  start() {
    this.isActive = true;
    window.WindVane.call('WVMotion', 'startCompass',
      { interval: 'ui' },
      (result) => this.update(result)
    );
  }

  update(result) {
    if (!this.isActive) return;
    
    // Xoay kim chỉ
    this.needle.style.transform = `rotate(${result.direction}deg)`;
    
    // Hiển thị info
    const compass = this.getDirection(result.direction);
    this.info.textContent = `${Math.round(result.direction)}° ${compass}`;
    
    // Highlight chữ N/S/E/W
    this.highlightCardinalDirection(result.direction);
  }

  getDirection(degrees) {
    const dirs = ['Bắc', 'Đông Bắc', 'Đông', 'Đông Nam', 
                  'Nam', 'Tây Nam', 'Tây', 'Tây Bắc'];
    return dirs[Math.round(degrees / 45) % 8];
  }

  highlightCardinalDirection(degrees) {
    // Remove all highlights
    document.querySelectorAll('.label').forEach(el => {
      el.style.color = 'black';
      el.style.fontSize = '14px';
    });
    
    // Highlight nearest cardinal direction
    if (degrees >= 337.5 || degrees < 22.5) {
      document.querySelector('.label.n').style.color = 'red';
      document.querySelector('.label.n').style.fontSize = '18px';
    } else if (degrees >= 67.5 && degrees < 112.5) {
      document.querySelector('.label.e').style.color = 'red';
      document.querySelector('.label.e').style.fontSize = '18px';
    } else if (degrees >= 157.5 && degrees < 202.5) {
      document.querySelector('.label.s').style.color = 'red';
      document.querySelector('.label.s').style.fontSize = '18px';
    } else if (degrees >= 247.5 && degrees < 292.5) {
      document.querySelector('.label.w').style.color = 'red';
      document.querySelector('.label.w').style.fontSize = '18px';
    }
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopCompass', {});
  }
}

const compass = new CompassUI();
compass.start();
```

### 3. AR Navigation

```javascript
class ARNavigation {
  constructor(targetLat, targetLng) {
    this.targetLat = targetLat;
    this.targetLng = targetLng;
    this.currentLocation = null;
    this.isActive = false;
  }

  start() {
    this.isActive = true;
    
    // Lấy vị trí hiện tại
    window.WindVane.call('WVLocation', 'getLocation', 
      { enableHighAccuracy: 'true' },
      (location) => {
        this.currentLocation = location.coords;
        this.startCompass();
      }
    );
  }

  startCompass() {
    window.WindVane.call('WVMotion', 'startCompass',
      { interval: 'game' },
      (result) => this.updateDirection(result)
    );
  }

  updateDirection(compassData) {
    if (!this.isActive || !this.currentLocation) return;
    
    // Tính bearing đến target
    const targetBearing = this.calculateBearing(
      this.currentLocation.latitude,
      this.currentLocation.longitude,
      this.targetLat,
      this.targetLng
    );
    
    // Tính góc lệch giữa hướng thiết bị và target
    let relativeBearing = targetBearing - compassData.direction;
    if (relativeBearing < 0) relativeBearing += 360;
    if (relativeBearing > 180) relativeBearing -= 360;
    
    // Tính khoảng cách
    const distance = this.calculateDistance(
      this.currentLocation.latitude,
      this.currentLocation.longitude,
      this.targetLat,
      this.targetLng
    );
    
    // Update UI
    this.updateARView(relativeBearing, distance);
  }

  calculateBearing(lat1, lon1, lat2, lon2) {
    const dLon = (lon2 - lon1) * Math.PI / 180;
    lat1 = lat1 * Math.PI / 180;
    lat2 = lat2 * Math.PI / 180;
    
    const y = Math.sin(dLon) * Math.cos(lat2);
    const x = Math.cos(lat1) * Math.sin(lat2) - 
              Math.sin(lat1) * Math.cos(lat2) * Math.cos(dLon);
    
    let bearing = Math.atan2(y, x) * 180 / Math.PI;
    return (bearing + 360) % 360;
  }

  calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371e3; // Earth radius in meters
    const φ1 = lat1 * Math.PI / 180;
    const φ2 = lat2 * Math.PI / 180;
    const Δφ = (lat2 - lat1) * Math.PI / 180;
    const Δλ = (lon2 - lon1) * Math.PI / 180;

    const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
              Math.cos(φ1) * Math.cos(φ2) *
              Math.sin(Δλ/2) * Math.sin(Δλ/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

    return R * c; // Distance in meters
  }

  updateARView(bearing, distance) {
    // Xoay mũi tên chỉ hướng
    const arrow = document.getElementById('ar-arrow');
    arrow.style.transform = `rotate(${bearing}deg)`;
    
    // Hiển thị khoảng cách
    const distText = distance > 1000 
      ? `${(distance / 1000).toFixed(1)} km`
      : `${Math.round(distance)} m`;
    
    document.getElementById('distance').textContent = distText;
    
    // Hiển thị hướng dẫn
    let instruction = '';
    if (Math.abs(bearing) < 10) {
      instruction = 'Đi thẳng';
    } else if (bearing > 0) {
      instruction = `Rẽ phải ${Math.round(bearing)}°`;
    } else {
      instruction = `Rẽ trái ${Math.round(Math.abs(bearing))}°`;
    }
    
    document.getElementById('instruction').textContent = instruction;
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopCompass', {});
  }
}

// Sử dụng
const nav = new ARNavigation(21.0285, 105.8542); // Hà Nội
nav.start();
```

### 4. Calibration check

```javascript
class CompassCalibration {
  constructor() {
    this.accuracyHistory = [];
    this.maxHistory = 10;
  }

  start() {
    window.WindVane.call('WVMotion', 'startCompass',
      { interval: 'ui' },
      (result) => this.checkAccuracy(result)
    );
  }

  checkAccuracy(result) {
    this.accuracyHistory.push(result.accuracy);
    
    if (this.accuracyHistory.length > this.maxHistory) {
      this.accuracyHistory.shift();
    }
    
    const avgAccuracy = this.accuracyHistory.reduce((a, b) => a + b, 0) 
                       / this.accuracyHistory.length;
    
    if (avgAccuracy > 20) { // degrees
      this.showCalibrationWarning();
    }
  }

  showCalibrationWarning() {
    const warning = document.createElement('div');
    warning.className = 'calibration-warning';
    warning.innerHTML = `
      <h3>⚠️ Compass cần hiệu chỉnh</h3>
      <p>Hãy xoay thiết bị theo hình số 8 trong không gian</p>
      <img src="/calibration-guide.gif" />
    `;
    document.body.appendChild(warning);
  }
}
```

### 5. React Hook

```jsx
import { useState, useEffect } from 'react';

function useCompass(interval = 'ui') {
  const [direction, setDirection] = useState(0);
  const [accuracy, setAccuracy] = useState(0);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    setIsActive(true);
    
    window.WindVane.call('WVMotion', 'startCompass',
      { interval },
      (result) => {
        setDirection(result.direction);
        setAccuracy(result.accuracy);
      },
      (error) => {
        console.error('Compass error:', error);
        setIsActive(false);
      }
    );

    return () => {
      window.WindVane.call('WVMotion', 'stopCompass', {});
      setIsActive(false);
    };
  }, [interval]);

  return { direction, accuracy, isActive };
}

// Component
function CompassDisplay() {
  const { direction, accuracy } = useCompass('ui');

  const getCardinalDirection = (deg) => {
    const dirs = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
    return dirs[Math.round(deg / 45) % 8];
  };

  return (
    <div>
      <h2>{Math.round(direction)}° {getCardinalDirection(direction)}</h2>
      <p>Độ chính xác: ±{accuracy}°</p>
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Calibration check**: Kiểm tra `accuracy` và yêu cầu calibration nếu > 20°
- **Smooth rotation**: Dùng CSS transition để làm mượt animation
- **Stop khi không dùng**: Gọi `stopCompass` để tiết kiệm pin
- **Combine với GPS**: Kết hợp compass + location cho navigation

### ❌ Không nên

- Quên stop compass (tốn pin)
- Update UI quá nhanh (gây lag)
- Không xử lý trường hợp compass không available
- Dựa vào compass trong môi trường có nhiễu từ

## Use Cases

| Use Case | Config | Note |
|----------|--------|------|
| **Navigation** | `interval: 'game'` | Kết hợp với GPS |
| **AR App** | `interval: 'game'` | Cần calibration tốt |
| **Map orientation** | `interval: 'ui'` | Xoay map theo hướng |
| **Photo geo-tagging** | `interval: 'normal'` | Chỉ cần hướng khi chụp |

## Giới hạn

- Chỉ hoạt động khi app ở foreground
- Bị ảnh hưởng bởi từ trường (case kim loại, loa, nam châm)
- Cần calibration định kỳ
- iOS yêu cầu HTTPS

## Xem thêm

- [Accelerometer](./accelerometer) - Cảm biến gia tốc
- [Gyroscope](./gyroscope) - Con quay hồi chuyển
- [getLocation](../F_media_location/getLocation) - GPS
