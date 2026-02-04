---
sidebar_position: 10
---

# Accelerometer

Truy cập cảm biến gia tốc (accelerometer) để đo chuyển động và độ nghiêng thiết bị.

## Cú pháp

```javascript
// Bắt đầu lắng nghe
window.WindVane.call(
  'WVMotion',
  'startAccelerometer',
  {
    interval: 'ui'  // 'game', 'ui', 'normal'
  },
  function(result) {
    // Callback được gọi liên tục
    console.log('X:', result.x);
    console.log('Y:', result.y);
    console.log('Z:', result.z);
  },
  function(error) {
    console.error('Accelerometer error:', error);
  }
);

// Dừng lắng nghe
window.WindVane.call('WVMotion', 'stopAccelerometer', {});
```

## Tham số đầu vào

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|-------|
| `interval` | `string` | `'normal'` | Tần suất update: `game` (60Hz), `ui` (30Hz), `normal` (10Hz) |

## Success Callback

Callback được gọi **liên tục** với tần suất tùy theo `interval`:

```typescript
{
  x: number,   // Gia tốc trục X (m/s²)
  y: number,   // Gia tốc trục Y (m/s²)
  z: number,   // Gia tốc trục Z (m/s²)
  timestamp: number  // Thời điểm đo (ms)
}
```

### Hệ tọa độ

```
         +Y (Hướng lên)
          |
          |
          |
          +------- +X (Hướng phải)
         /
        /
       +Z (Hướng ra ngoài màn hình)
```

## Ví dụ

### 1. Phát hiện lắc thiết bị (shake detection)

```javascript
class ShakeDetector {
  constructor(threshold = 15, minShakes = 3) {
    this.threshold = threshold;
    this.minShakes = minShakes;
    this.shakes = 0;
    this.lastShakeTime = 0;
    this.isListening = false;
  }

  start(onShake) {
    this.onShake = onShake;
    this.isListening = true;
    
    window.WindVane.call('WVMotion', 'startAccelerometer', {
      interval: 'ui'  // 30Hz đủ cho shake detection
    }, (data) => {
      this.handleAccelerometer(data);
    });
  }

  handleAccelerometer(data) {
    if (!this.isListening) return;
    
    // Tính độ lớn gia tốc
    const magnitude = Math.sqrt(
      data.x * data.x + 
      data.y * data.y + 
      data.z * data.z
    );
    
    // Bỏ qua gravity (9.8 m/s²)
    const acceleration = Math.abs(magnitude - 9.8);
    
    // Phát hiện shake
    if (acceleration > this.threshold) {
      const now = Date.now();
      
      // Reset nếu quá lâu giữa các lần shake
      if (now - this.lastShakeTime > 1000) {
        this.shakes = 0;
      }
      
      this.shakes++;
      this.lastShakeTime = now;
      
      // Trigger callback nếu đủ số lần shake
      if (this.shakes >= this.minShakes) {
        this.shakes = 0;
        if (this.onShake) {
          this.onShake();
        }
      }
    }
  }

  stop() {
    this.isListening = false;
    window.WindVane.call('WVMotion', 'stopAccelerometer', {});
  }
}

// Sử dụng
const shakeDetector = new ShakeDetector(15, 3);
shakeDetector.start(() => {
  console.log('Device shaken!');
  window.WindVane.call('WVUIToast', 'toast', {
    message: 'Đã phát hiện lắc thiết bị!'
  });
});
```

### 2. Tilt sensor (cảm biến nghiêng)

```javascript
class TiltSensor {
  constructor() {
    this.isListening = false;
  }

  start(onTilt) {
    this.onTilt = onTilt;
    this.isListening = true;
    
    window.WindVane.call('WVMotion', 'startAccelerometer', {
      interval: 'ui'
    }, (data) => {
      this.handleAccelerometer(data);
    });
  }

  handleAccelerometer(data) {
    if (!this.isListening) return;
    
    // Tính góc nghiêng (degrees)
    const tiltX = Math.atan2(data.y, data.z) * (180 / Math.PI);
    const tiltY = Math.atan2(data.x, data.z) * (180 / Math.PI);
    
    // Xác định hướng nghiêng
    let direction = 'flat';
    if (Math.abs(tiltX) > 30) {
      direction = tiltX > 0 ? 'forward' : 'backward';
    }
    if (Math.abs(tiltY) > 30) {
      direction = tiltY > 0 ? 'right' : 'left';
    }
    
    if (this.onTilt) {
      this.onTilt({
        tiltX: tiltX.toFixed(1),
        tiltY: tiltY.toFixed(1),
        direction: direction
      });
    }
  }

  stop() {
    this.isListening = false;
    window.WindVane.call('WVMotion', 'stopAccelerometer', {});
  }
}

// Sử dụng cho game điều khiển bằng nghiêng
const tiltSensor = new TiltSensor();
tiltSensor.start((tilt) => {
  console.log(`Nghiêng: ${tilt.direction}, X: ${tilt.tiltX}°, Y: ${tilt.tiltY}°`);
  moveGameCharacter(tilt.direction);
});
```

### 3. Pedometer (đếm bước chân)

```javascript
class Pedometer {
  constructor() {
    this.steps = 0;
    this.lastMagnitude = 0;
    this.lastStepTime = 0;
    this.isListening = false;
  }

  start() {
    this.isListening = true;
    
    window.WindVane.call('WVMotion', 'startAccelerometer', {
      interval: 'normal'  // 10Hz đủ cho đếm bước
    }, (data) => {
      this.detectStep(data);
    });
  }

  detectStep(data) {
    if (!this.isListening) return;
    
    // Tính độ lớn vector gia tốc
    const magnitude = Math.sqrt(
      data.x * data.x + 
      data.y * data.y + 
      data.z * data.z
    );
    
    const now = Date.now();
    
    // Phát hiện peak (đỉnh)
    const threshold = 11.0;  // m/s²
    const minStepInterval = 200;  // ms (5 bước/giây tối đa)
    
    if (magnitude > threshold && 
        this.lastMagnitude < threshold &&
        now - this.lastStepTime > minStepInterval) {
      
      this.steps++;
      this.lastStepTime = now;
      this.onStep(this.steps);
    }
    
    this.lastMagnitude = magnitude;
  }

  onStep(totalSteps) {
    // Update UI
    document.getElementById('steps').textContent = totalSteps;
    
    // Tính khoảng cách và calories
    const distanceKm = totalSteps * 0.0007;  // ~70cm/bước
    const calories = distanceKm * 65;  // ~65 cal/km
    
    document.getElementById('distance').textContent = 
      distanceKm.toFixed(2) + ' km';
    document.getElementById('calories').textContent = 
      Math.floor(calories) + ' cal';
  }

  stop() {
    this.isListening = false;
    window.WindVane.call('WVMotion', 'stopAccelerometer', {});
  }

  reset() {
    this.steps = 0;
    this.lastMagnitude = 0;
    this.lastStepTime = 0;
  }
}
```

### 4. Game controller

```javascript
class AccelerometerGameController {
  constructor(sensitivity = 1.0) {
    this.sensitivity = sensitivity;
    this.isActive = false;
  }

  start(onMove) {
    this.onMove = onMove;
    this.isActive = true;
    
    window.WindVane.call('WVMotion', 'startAccelerometer', {
      interval: 'game'  // 60Hz cho game mượt mà
    }, (data) => {
      this.handleInput(data);
    });
  }

  handleInput(data) {
    if (!this.isActive) return;
    
    // Normalize và áp dụng sensitivity
    const moveX = data.x * this.sensitivity;
    const moveY = data.y * this.sensitivity;
    
    // Deadzone để tránh drift
    const deadzone = 0.5;
    const finalX = Math.abs(moveX) > deadzone ? moveX : 0;
    const finalY = Math.abs(moveY) > deadzone ? moveY : 0;
    
    if (this.onMove) {
      this.onMove(finalX, finalY);
    }
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopAccelerometer', {});
  }

  setSensitivity(value) {
    this.sensitivity = value;
  }
}

// Sử dụng trong game
const controller = new AccelerometerGameController(1.5);
controller.start((x, y) => {
  // Di chuyển nhân vật game
  player.x += x * 2;
  player.y += y * 2;
  
  // Giới hạn trong canvas
  player.x = Math.max(0, Math.min(canvas.width, player.x));
  player.y = Math.max(0, Math.min(canvas.height, player.y));
});
```

### 5. React Hook

```jsx
import { useState, useEffect, useRef } from 'react';

function useAccelerometer(interval = 'ui', enabled = true) {
  const [data, setData] = useState({ x: 0, y: 0, z: 0 });
  const [isActive, setIsActive] = useState(false);
  const callbackRef = useRef();

  useEffect(() => {
    if (!enabled) return;

    setIsActive(true);
    
    window.WindVane.call('WVMotion', 'startAccelerometer',
      { interval },
      (result) => {
        setData(result);
        if (callbackRef.current) {
          callbackRef.current(result);
        }
      },
      (error) => {
        console.error('Accelerometer error:', error);
        setIsActive(false);
      }
    );

    return () => {
      window.WindVane.call('WVMotion', 'stopAccelerometer', {});
      setIsActive(false);
    };
  }, [interval, enabled]);

  const setCallback = (callback) => {
    callbackRef.current = callback;
  };

  return { data, isActive, setCallback };
}

// Component sử dụng
function ShakeToRefresh() {
  const [shakeCount, setShakeCount] = useState(0);
  const lastShakeRef = useRef(0);

  const { data } = useAccelerometer('ui', true);

  useEffect(() => {
    const magnitude = Math.sqrt(
      data.x * data.x + 
      data.y * data.y + 
      data.z * data.z
    );
    
    if (Math.abs(magnitude - 9.8) > 15) {
      const now = Date.now();
      if (now - lastShakeRef.current > 500) {
        setShakeCount(c => c + 1);
        lastShakeRef.current = now;
        
        // Refresh data
        fetchData();
      }
    }
  }, [data]);

  return (
    <div>
      <h3>Shake to Refresh</h3>
      <p>Lắc thiết bị để làm mới dữ liệu</p>
      <p>Số lần lắc: {shakeCount}</p>
      <p>X: {data.x.toFixed(2)}, Y: {data.y.toFixed(2)}, Z: {data.z.toFixed(2)}</p>
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Stop khi không dùng**: Luôn gọi `stopAccelerometer` để tiết kiệm pin
- **Chọn interval phù hợp**: `game` cho game, `ui` cho UI, `normal` cho tracking
- **Deadzone**: Áp dụng deadzone để tránh jitter
- **Throttle**: Throttle callback nếu không cần update quá nhanh
- **Battery aware**: Giảm frequency khi pin yếu

### ❌ Không nên

- Quên stop accelerometer (tốn pin)
- Dùng `game` interval cho non-game apps
- Xử lý logic nặng trong callback
- Update UI quá thường xuyên (gây lag)

## Use Cases

| Use Case | Interval | Logic |
|----------|----------|-------|
| **Shake to refresh** | `ui` | Detect magnitude spike |
| **Tilt game control** | `game` | Calculate tilt angles |
| **Pedometer** | `normal` | Peak detection |
| **Compass calibration** | `ui` | Detect rotation |

## Battery Impact

| Interval | Frequency | Battery Impact |
|----------|-----------|----------------|
| `game` | 60Hz | 🔴 Cao |
| `ui` | 30Hz | 🟡 Trung bình |
| `normal` | 10Hz | 🟢 Thấp |

## Giới hạn

- Chỉ hoạt động khi app ở foreground
- Accuracy phụ thuộc vào hardware
- Không hỗ trợ background monitoring

## Xem thêm

- [Compass](./compass) - La bàn điện tử
- [Gyroscope](./gyroscope) - Con quay hồi chuyển
