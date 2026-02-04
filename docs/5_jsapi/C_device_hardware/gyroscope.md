---
sidebar_position: 12
---

# Gyroscope

Truy cập con quay hồi chuyển (gyroscope) để đo tốc độ xoay của thiết bị.

## Cú pháp

```javascript
// Bắt đầu lắng nghe
window.WindVane.call('WVMotion', 'startGyroscope', params, successCallback, failCallback);

// Dừng lắng nghe
window.WindVane.call('WVMotion', 'stopGyroscope', {});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `interval` | `string` | Không | `'game'`: 60Hz<br/>`'ui'`: 30Hz (mặc định)<br/>`'normal'`: 10Hz |

## Success Callback

```javascript
{
  x: number,          // Tốc độ xoay quanh trục X (rad/s)
  y: number,          // Tốc độ xoay quanh trục Y (rad/s)
  z: number,          // Tốc độ xoay quanh trục Z (rad/s)
  timestamp: number   // Thời điểm đo (ms)
}
```

### Hệ tọa độ

```
         +Y (Hướng lên)
          |
          |
          +------- +X (Hướng phải)
         /
        /
       +Z (Hướng ra ngoài màn hình)

Xoay dương: Ngược chiều kim đồng hồ khi nhìn theo hướng dương của trục
```

## Ví dụ

### 1. Phát hiện xoay thiết bị

```javascript
class RotationDetector {
  constructor(threshold = 2.0) {
    this.threshold = threshold; // rad/s
    this.isActive = false;
  }

  start(onRotate) {
    this.onRotate = onRotate;
    this.isActive = true;
    
    window.WindVane.call('WVMotion', 'startGyroscope',
      { interval: 'ui' },
      (data) => this.handleGyro(data)
    );
  }

  handleGyro(data) {
    if (!this.isActive) return;
    
    // Tính tổng tốc độ xoay
    const totalRotation = Math.sqrt(
      data.x * data.x + 
      data.y * data.y + 
      data.z * data.z
    );
    
    if (totalRotation > this.threshold) {
      // Xác định trục xoay chính
      const absX = Math.abs(data.x);
      const absY = Math.abs(data.y);
      const absZ = Math.abs(data.z);
      
      let axis = 'x';
      let maxRotation = absX;
      
      if (absY > maxRotation) {
        axis = 'y';
        maxRotation = absY;
      }
      if (absZ > maxRotation) {
        axis = 'z';
        maxRotation = absZ;
      }
      
      const direction = data[axis] > 0 ? 'positive' : 'negative';
      
      if (this.onRotate) {
        this.onRotate({
          axis: axis,
          direction: direction,
          speed: maxRotation
        });
      }
    }
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopGyroscope', {});
  }
}

// Sử dụng
const detector = new RotationDetector(2.0);
detector.start((rotation) => {
  console.log(`Xoay ${rotation.axis} hướng ${rotation.direction}`);
  console.log(`Tốc độ: ${rotation.speed.toFixed(2)} rad/s`);
});
```

### 2. 3D Object Controller

```javascript
class Object3DController {
  constructor(object3D) {
    this.object = object3D;
    this.sensitivity = 1.0;
    this.isActive = false;
    this.rotationX = 0;
    this.rotationY = 0;
    this.rotationZ = 0;
  }

  start() {
    this.isActive = true;
    
    window.WindVane.call('WVMotion', 'startGyroscope',
      { interval: 'game' }, // 60Hz cho animation mượt
      (data) => this.updateRotation(data)
    );
    
    this.animate();
  }

  updateRotation(data) {
    if (!this.isActive) return;
    
    // Tích lũy góc xoay (integration)
    const dt = 0.016; // ~60fps
    this.rotationX += data.x * dt * this.sensitivity;
    this.rotationY += data.y * dt * this.sensitivity;
    this.rotationZ += data.z * dt * this.sensitivity;
  }

  animate() {
    if (!this.isActive) return;
    
    // Apply rotation to 3D object (Three.js example)
    this.object.rotation.x = this.rotationX;
    this.object.rotation.y = this.rotationY;
    this.object.rotation.z = this.rotationZ;
    
    requestAnimationFrame(() => this.animate());
  }

  reset() {
    this.rotationX = 0;
    this.rotationY = 0;
    this.rotationZ = 0;
  }

  setSensitivity(value) {
    this.sensitivity = value;
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopGyroscope', {});
  }
}

// Sử dụng với Three.js
const cube = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);

const controller = new Object3DController(cube);
controller.start();
```

### 3. Panorama Viewer

```javascript
class PanoramaViewer {
  constructor(imageUrl) {
    this.imageUrl = imageUrl;
    this.yaw = 0;   // Xoay trái-phải (Y axis)
    this.pitch = 0; // Xoay lên-xuống (X axis)
    this.isActive = false;
  }

  start() {
    this.isActive = true;
    this.loadPanorama();
    
    window.WindVane.call('WVMotion', 'startGyroscope',
      { interval: 'game' },
      (data) => this.updateView(data)
    );
  }

  loadPanorama() {
    // Load 360° image
    const img = new Image();
    img.onload = () => {
      this.panoramaImage = img;
      this.drawPanorama();
    };
    img.src = this.imageUrl;
  }

  updateView(data) {
    if (!this.isActive) return;
    
    const dt = 0.016; // ~60fps
    
    // Update yaw (trái-phải)
    this.yaw += data.y * dt * 0.5;
    
    // Update pitch (lên-xuống)
    this.pitch += data.x * dt * 0.5;
    
    // Giới hạn pitch (-90° đến +90°)
    this.pitch = Math.max(-Math.PI/2, Math.min(Math.PI/2, this.pitch));
    
    // Yaw không giới hạn (360° quay tròn)
    if (this.yaw > Math.PI) this.yaw -= 2 * Math.PI;
    if (this.yaw < -Math.PI) this.yaw += 2 * Math.PI;
    
    this.drawPanorama();
  }

  drawPanorama() {
    const canvas = document.getElementById('panorama-canvas');
    const ctx = canvas.getContext('2d');
    
    if (!this.panoramaImage) return;
    
    // Calculate viewport position in panorama
    const imgWidth = this.panoramaImage.width;
    const imgHeight = this.panoramaImage.height;
    
    // Convert yaw/pitch to image coordinates
    const centerX = (this.yaw / (2 * Math.PI) + 0.5) * imgWidth;
    const centerY = (this.pitch / Math.PI + 0.5) * imgHeight;
    
    // Draw visible portion
    const viewWidth = canvas.width;
    const viewHeight = canvas.height;
    
    ctx.drawImage(
      this.panoramaImage,
      centerX - viewWidth/2, centerY - viewHeight/2, viewWidth, viewHeight,
      0, 0, viewWidth, viewHeight
    );
    
    // Draw crosshair
    this.drawCrosshair(ctx, canvas.width/2, canvas.height/2);
  }

  drawCrosshair(ctx, x, y) {
    ctx.strokeStyle = 'white';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(x - 20, y);
    ctx.lineTo(x + 20, y);
    ctx.moveTo(x, y - 20);
    ctx.lineTo(x, y + 20);
    ctx.stroke();
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopGyroscope', {});
  }
}

// Sử dụng
const viewer = new PanoramaViewer('/panorama.jpg');
viewer.start();
```

### 4. Gesture Recognition

```javascript
class GyroGestureRecognizer {
  constructor() {
    this.gestures = {
      shake: { threshold: 5.0, duration: 200 },
      tilt: { threshold: 2.0, duration: 500 },
      spin: { threshold: 3.0, duration: 300 }
    };
    this.history = [];
    this.maxHistory = 30; // 0.5s @ 60Hz
  }

  start(onGesture) {
    this.onGesture = onGesture;
    this.isActive = true;
    
    window.WindVane.call('WVMotion', 'startGyroscope',
      { interval: 'game' },
      (data) => this.detectGesture(data)
    );
  }

  detectGesture(data) {
    if (!this.isActive) return;
    
    this.history.push({
      x: data.x,
      y: data.y,
      z: data.z,
      timestamp: data.timestamp
    });
    
    if (this.history.length > this.maxHistory) {
      this.history.shift();
    }
    
    // Detect shake
    if (this.detectShake()) {
      this.onGesture({ type: 'shake' });
      this.history = [];
      return;
    }
    
    // Detect spin
    if (this.detectSpin()) {
      this.onGesture({ type: 'spin' });
      this.history = [];
      return;
    }
  }

  detectShake() {
    if (this.history.length < 10) return false;
    
    let peaks = 0;
    for (let i = 1; i < this.history.length - 1; i++) {
      const curr = this.history[i];
      const prev = this.history[i - 1];
      const next = this.history[i + 1];
      
      const magnitude = Math.sqrt(curr.x**2 + curr.y**2 + curr.z**2);
      const prevMag = Math.sqrt(prev.x**2 + prev.y**2 + prev.z**2);
      const nextMag = Math.sqrt(next.x**2 + next.y**2 + next.z**2);
      
      if (magnitude > this.gestures.shake.threshold &&
          magnitude > prevMag && magnitude > nextMag) {
        peaks++;
      }
    }
    
    return peaks >= 3;
  }

  detectSpin() {
    if (this.history.length < 15) return false;
    
    // Check for sustained rotation around Z axis
    let totalZ = 0;
    for (const point of this.history) {
      totalZ += Math.abs(point.z);
    }
    
    const avgZ = totalZ / this.history.length;
    return avgZ > this.gestures.spin.threshold;
  }

  stop() {
    this.isActive = false;
    window.WindVane.call('WVMotion', 'stopGyroscope', {});
  }
}

// Sử dụng
const recognizer = new GyroGestureRecognizer();
recognizer.start((gesture) => {
  console.log('Detected gesture:', gesture.type);
  
  if (gesture.type === 'shake') {
    refreshContent();
  } else if (gesture.type === 'spin') {
    rotateView360();
  }
});
```

### 5. React Hook

```jsx
import { useState, useEffect, useRef } from 'react';

function useGyroscope(interval = 'ui') {
  const [rotation, setRotation] = useState({ x: 0, y: 0, z: 0 });
  const [isActive, setIsActive] = useState(false);
  const integratedRef = useRef({ x: 0, y: 0, z: 0 });

  useEffect(() => {
    setIsActive(true);
    
    window.WindVane.call('WVMotion', 'startGyroscope',
      { interval },
      (result) => {
        setRotation(result);
        
        // Integrate rotation over time
        const dt = interval === 'game' ? 0.016 : 0.033;
        integratedRef.current.x += result.x * dt;
        integratedRef.current.y += result.y * dt;
        integratedRef.current.z += result.z * dt;
      },
      (error) => {
        console.error('Gyroscope error:', error);
        setIsActive(false);
      }
    );

    return () => {
      window.WindVane.call('WVMotion', 'stopGyroscope', {});
      setIsActive(false);
    };
  }, [interval]);

  const reset = () => {
    integratedRef.current = { x: 0, y: 0, z: 0 };
  };

  return {
    rotation,           // Current rotation speed
    integrated: integratedRef.current,  // Accumulated rotation
    isActive,
    reset
  };
}

// Component
function GyroscopeVisualizer() {
  const { rotation, integrated, reset } = useGyroscope('game');

  return (
    <div>
      <h3>Rotation Speed (rad/s)</h3>
      <p>X: {rotation.x.toFixed(3)}</p>
      <p>Y: {rotation.y.toFixed(3)}</p>
      <p>Z: {rotation.z.toFixed(3)}</p>
      
      <h3>Total Rotation (rad)</h3>
      <p>X: {integrated.x.toFixed(2)}</p>
      <p>Y: {integrated.y.toFixed(2)}</p>
      <p>Z: {integrated.z.toFixed(2)}</p>
      
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Integration**: Tích phân tốc độ xoay để có góc xoay tổng
- **Complementary filter**: Kết hợp với Accelerometer để giảm drift
- **Deadzone**: Bỏ qua giá trị nhỏ (< 0.1 rad/s) để tránh noise
- **Battery aware**: Dùng `normal` interval khi không cần độ chính xác cao
- **Stop khi không dùng**: Tiết kiệm pin

### ❌ Không nên

- Quên stop gyroscope (tốn pin)
- Dùng `game` interval cho non-game apps
- Không xử lý gyro drift (tích phân lâu dài sẽ sai lệch)
- Update UI quá nhanh

## Use Cases

| Use Case | Interval | Note |
|----------|----------|------|
| **3D Game** | `game` | Cần độ chính xác cao |
| **Panorama** | `game` | Smooth view rotation |
| **Gesture** | `ui` | Đủ cho detection |
| **Shake detect** | `ui` | Không cần quá nhanh |

## Gyro Drift

Gyroscope bị **drift** khi tích phân lâu:

```javascript
// Không tốt: Drift tích lũy
let totalRotation = 0;
gyroscope.on('data', (data) => {
  totalRotation += data.z * dt; // Sai lệch dần
});

// Tốt hơn: Reset định kỳ
let totalRotation = 0;
let lastReset = Date.now();

gyroscope.on('data', (data) => {
  totalRotation += data.z * dt;
  
  // Reset mỗi 10s
  if (Date.now() - lastReset > 10000) {
    totalRotation = 0;
    lastReset = Date.now();
  }
});

// Tốt nhất: Kết hợp Accelerometer + Compass
// (Complementary Filter)
```

## Giới hạn

- Drift sau ~10-30s tích phân
- Chỉ hoạt động khi app foreground
- Không có gyroscope trên một số thiết bị cũ
- Battery impact với `game` interval

## Xem thêm

- [Accelerometer](./accelerometer) - Cảm biến gia tốc
- [Compass](./compass) - La bàn điện tử
