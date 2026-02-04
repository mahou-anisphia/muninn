---
sidebar_position: 12
---

# Motion startGyroscope - Bắt đầu đọc gyroscope

Bật cảm biến gyroscope để đo tốc độ góc (rotation rate).

## API Call

```javascript
window.WindVane.call('WVMotion', 'startGyroscope', {
  interval: 'game'
}, () => {
  console.log('Gyroscope started');
  
  // Listen for data
  window.WindVane.call('WVMotion', 'onGyroscopeChange', {}, (data) => {
    console.log('Rotation X:', data.x);
    console.log('Rotation Y:', data.y);
    console.log('Rotation Z:', data.z);
  });
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `interval` | `string` | Không | Tần suất đọc: `'ui'`, `'game'`, `'normal'` |

### Intervals

| Interval | Tần suất | Use case |
|----------|---------|----------|
| `ui` | ~60Hz | Smooth UI animations |
| `game` | ~20Hz | Game controls |
| `normal` | ~5Hz | Normal monitoring |

## Gyroscope Data

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `x` | `number` | Tốc độ quay quanh trục X (rad/s) |
| `y` | `number` | Tốc độ quay quanh trục Y (rad/s) |
| `z` | `number` | Tốc độ quay quanh trục Z (rad/s) |
| `timestamp` | `number` | Thời gian đo (ms) |

## Use Case: Gyro-based Game Controls

```javascript
class GyroGameController {
  constructor() {
    this.sensitivity = 0.5;
    this.deadzone = 0.1; // rad/s
  }

  start() {
    window.WindVane.call('WVMotion', 'startGyroscope', {
      interval: 'game'
    }, () => {
      window.WindVane.call('WVMotion', 'onGyroscopeChange', {}, (data) => {
        this.updateControls(data);
      });
    });
  }

  updateControls(data) {
    // Apply deadzone
    const x = Math.abs(data.x) > this.deadzone ? data.x : 0;
    const y = Math.abs(data.y) > this.deadzone ? data.y : 0;

    // Map to game controls
    const tilt = {
      horizontal: -y * this.sensitivity,
      vertical: x * this.sensitivity
    };

    this.movePlayer(tilt);
  }

  movePlayer(tilt) {
    const player = document.getElementById('player');
    
    if (player) {
      const currentLeft = parseFloat(player.style.left || 50);
      const currentTop = parseFloat(player.style.top || 50);

      player.style.left = `${currentLeft + tilt.horizontal}%`;
      player.style.top = `${currentTop + tilt.vertical}%`;
    }
  }

  stop() {
    window.WindVane.call('WVMotion', 'stopGyroscope', {});
  }
}

// Sử dụng
const gameController = new GyroGameController();
gameController.start();
```

## API liên quan

- [stopGyroscope](./motion_stopGyroscope) - Dừng gyroscope
- [compass](./compass) - Cảm biến la bàn
- [accelerometer](./accelerometer) - Gia tốc kế
