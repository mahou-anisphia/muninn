---
sidebar_position: 14
---

# Motion startCompass - Bắt đầu đọc la bàn

Bật cảm biến la bàn (compass) để đo hướng.

## API Call

```javascript
window.WindVane.call('WVMotion', 'startCompass', {}, () => {
  console.log('Compass started');
  
  window.WindVane.call('WVMotion', 'onCompassChange', {}, (data) => {
    console.log('Direction:', data.direction);
    console.log('Accuracy:', data.accuracy);
  });
});
```

## Compass Data

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `direction` | `number` | Hướng so với Bắc (0-360 độ) |
| `accuracy` | `string` | Độ chính xác: `'high'`, `'medium'`, `'low'`, `'unreliable'` |

## Use Case: Direction Indicator

```javascript
class DirectionIndicator {
  constructor() {
    this.currentDirection = 0;
  }

  start() {
    window.WindVane.call('WVMotion', 'startCompass', {}, () => {
      window.WindVane.call('WVMotion', 'onCompassChange', {}, (data) => {
        this.updateDirection(data);
      });
    });
  }

  updateDirection(data) {
    this.currentDirection = data.direction;
    
    const compass = document.getElementById('compass-needle');
    if (compass) {
      compass.style.transform = `rotate(${data.direction}deg)`;
    }

    this.updateLabel(data.direction);
  }

  updateLabel(direction) {
    const label = this.getDirectionLabel(direction);
    
    const dirLabel = document.getElementById('direction-label');
    if (dirLabel) {
      dirLabel.textContent = `${label} (${Math.round(direction)}°)`;
    }
  }

  getDirectionLabel(degrees) {
    const directions = [
      'Bắc', 'ĐB', 'Đông', 'ĐN',
      'Nam', 'TN', 'Tây', 'TB'
    ];
    
    const index = Math.round(degrees / 45) % 8;
    return directions[index];
  }

  stop() {
    window.WindVane.call('WVMotion', 'stopCompass', {});
  }
}

// Sử dụng
const indicator = new DirectionIndicator();
indicator.start();
```

## API liên quan

- [stopCompass](./motion_stopCompass) - Dừng la bàn
- [getLocation](../F_media_location/getLocation) - GPS location
