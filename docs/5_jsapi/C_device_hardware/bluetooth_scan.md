---
sidebar_position: 20
---

# Bluetooth: scan

Quét các thiết bị Bluetooth Low Energy (BLE) xung quanh.

## Cú pháp

```javascript
window.WindVane.call('WVBluetooth', 'scan', params, successCallback, failCallback);
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `services` | `string[]` | Không | Mảng UUID services cần quét (lọc theo service) |
| `allowDuplicates` | `boolean` | Không | Cho phép báo cáo thiết bị nhiều lần |
| `interval` | `number` | Không | Thời gian quét (ms), mặc định 10000 |

## Success Callback

Callback được gọi **mỗi khi phát hiện thiết bị**:

```javascript
{
  deviceId: string,       // Device ID (MAC address)
  name: string,           // Tên thiết bị
  RSSI: number,           // Cường độ tín hiệu (dBm)
  advertisData: object,   // Raw advertisement data
  serviceUUIDs: string[]  // Danh sách services
}
```

## Ví dụ

### 1. Quét thiết bị BLE cơ bản

```javascript
const discoveredDevices = new Map();

window.WindVane.call('WVBluetooth', 'scan',
  { interval: 10000 }, // Quét trong 10 giây
  (device) => {
    // Callback được gọi mỗi khi phát hiện thiết bị
    console.log('Phát hiện:', device.name, device.RSSI + 'dBm');
    
    // Lưu device (tránh duplicate)
    if (!discoveredDevices.has(device.deviceId)) {
      discoveredDevices.set(device.deviceId, device);
      displayDevice(device);
    }
  },
  (error) => {
    console.error('Scan error:', error);
  }
);

function displayDevice(device) {
  const list = document.getElementById('device-list');
  const item = document.createElement('div');
  item.className = 'device-item';
  item.innerHTML = `
    <h4>${device.name || 'Unnamed'}</h4>
    <p>ID: ${device.deviceId}</p>
    <p>Signal: ${device.RSSI} dBm</p>
  `;
  item.onclick = () => connectToDevice(device.deviceId);
  list.appendChild(item);
}
```

### 2. Quét theo service UUID (lọc thiết bị)

```javascript
// Chỉ quét thiết bị có Heart Rate service
const HEART_RATE_SERVICE = '0000180d-0000-1000-8000-00805f9b34fb';

window.WindVane.call('WVBluetooth', 'scan',
  { 
    services: [HEART_RATE_SERVICE],
    interval: 15000
  },
  (device) => {
    console.log('Heart Rate device:', device.name);
    // Chỉ nhận thiết bị có Heart Rate service
  },
  (error) => {
    console.error('Scan error:', error);
  }
);
```

### 3. Scanner class với UI

```javascript
class BLEScanner {
  constructor() {
    this.devices = new Map();
    this.isScanning = false;
    this.scanTimer = null;
  }

  start(duration = 10000) {
    if (this.isScanning) return;
    
    this.isScanning = true;
    this.devices.clear();
    this.updateUI('Đang quét...');
    
    window.WindVane.call('WVBluetooth', 'scan',
      { interval: duration, allowDuplicates: false },
      (device) => this.onDeviceFound(device),
      (error) => this.onError(error)
    );
    
    // Auto stop sau duration
    this.scanTimer = setTimeout(() => {
      this.stop();
    }, duration);
  }

  onDeviceFound(device) {
    // Filter thiết bị theo RSSI (chỉ lấy thiết bị gần)
    if (device.RSSI < -80) return;
    
    const existing = this.devices.get(device.deviceId);
    
    if (existing) {
      // Update RSSI nếu thiết bị đã có
      existing.RSSI = device.RSSI;
      this.updateDeviceRSSI(device.deviceId, device.RSSI);
    } else {
      // Thêm thiết bị mới
      this.devices.set(device.deviceId, device);
      this.addDeviceToUI(device);
    }
    
    this.updateUI(`Tìm thấy ${this.devices.size} thiết bị`);
  }

  addDeviceToUI(device) {
    const list = document.getElementById('device-list');
    const item = document.createElement('div');
    item.id = `device-${device.deviceId}`;
    item.className = 'device-item';
    item.innerHTML = `
      <div class="device-info">
        <h4>${device.name || 'Unnamed Device'}</h4>
        <p class="device-id">${device.deviceId}</p>
        <p class="device-rssi">Signal: ${device.RSSI} dBm</p>
      </div>
      <button onclick="bleScanner.connect('${device.deviceId}')">
        Kết nối
      </button>
    `;
    
    // Visual feedback dựa trên RSSI
    if (device.RSSI > -60) {
      item.classList.add('signal-excellent');
    } else if (device.RSSI > -70) {
      item.classList.add('signal-good');
    } else {
      item.classList.add('signal-weak');
    }
    
    list.appendChild(item);
  }

  updateDeviceRSSI(deviceId, rssi) {
    const elem = document.querySelector(`#device-${deviceId} .device-rssi`);
    if (elem) {
      elem.textContent = `Signal: ${rssi} dBm`;
    }
  }

  updateUI(message) {
    document.getElementById('scan-status').textContent = message;
  }

  connect(deviceId) {
    this.stop();
    const device = this.devices.get(deviceId);
    if (device) {
      console.log('Connecting to:', device.name);
      // See bluetooth_connect.md
      connectToDevice(deviceId);
    }
  }

  stop() {
    if (!this.isScanning) return;
    
    this.isScanning = false;
    clearTimeout(this.scanTimer);
    
    window.WindVane.call('WVBluetooth', 'stopScan', {});
    this.updateUI(`Quét xong. Tìm thấy ${this.devices.size} thiết bị.`);
  }

  onError(error) {
    this.isScanning = false;
    clearTimeout(this.scanTimer);
    
    if (error.error === 'NO_PERMISSION') {
      alert('Vui lòng cấp quyền Bluetooth trong Settings');
    } else if (error.error === 'BLUETOOTH_DISABLED') {
      alert('Vui lòng bật Bluetooth');
    } else {
      alert('Lỗi quét: ' + error.errorMessage);
    }
  }
}

// Sử dụng
const bleScanner = new BLEScanner();
bleScanner.start(15000);
```

### 4. Proximity detector

```javascript
class ProximityDetector {
  constructor(targetDeviceId, threshold = -70) {
    this.targetDeviceId = targetDeviceId;
    this.threshold = threshold; // dBm
    this.isNear = false;
  }

  start(onProximityChange) {
    this.onProximityChange = onProximityChange;
    
    window.WindVane.call('WVBluetooth', 'scan',
      { 
        interval: 60000,        // Quét liên tục
        allowDuplicates: true   // Nhận update RSSI
      },
      (device) => this.checkProximity(device)
    );
  }

  checkProximity(device) {
    if (device.deviceId !== this.targetDeviceId) return;
    
    const wasNear = this.isNear;
    this.isNear = device.RSSI > this.threshold;
    
    // Trigger callback khi thay đổi state
    if (wasNear !== this.isNear) {
      if (this.onProximityChange) {
        this.onProximityChange(this.isNear, device.RSSI);
      }
    }
  }

  stop() {
    window.WindVane.call('WVBluetooth', 'stopScan', {});
  }
}

// Sử dụng
const tracker = new ProximityDetector('AA:BB:CC:DD:EE:FF', -65);
tracker.start((isNear, rssi) => {
  if (isNear) {
    console.log('Thiết bị gần, RSSI:', rssi);
    showNotification('Đã tìm thấy thiết bị của bạn!');
  } else {
    console.log('Thiết bị xa, RSSI:', rssi);
    showNotification('Thiết bị đã rời xa');
  }
});
```

### 5. React Component

```jsx
import { useState, useEffect } from 'react';

function BLEDeviceList() {
  const [devices, setDevices] = useState([]);
  const [isScanning, setIsScanning] = useState(false);

  const startScan = () => {
    setIsScanning(true);
    setDevices([]);

    window.WindVane.call('WVBluetooth', 'scan',
      { interval: 15000 },
      (device) => {
        setDevices(prev => {
          const index = prev.findIndex(d => d.deviceId === device.deviceId);
          if (index >= 0) {
            // Update existing
            const updated = [...prev];
            updated[index] = device;
            return updated;
          } else {
            // Add new
            return [...prev, device];
          }
        });
      },
      (error) => {
        console.error('Scan error:', error);
        setIsScanning(false);
      }
    );

    // Auto stop
    setTimeout(() => {
      window.WindVane.call('WVBluetooth', 'stopScan', {});
      setIsScanning(false);
    }, 15000);
  };

  const stopScan = () => {
    window.WindVane.call('WVBluetooth', 'stopScan', {});
    setIsScanning(false);
  };

  return (
    <div>
      <div className="controls">
        <button onClick={startScan} disabled={isScanning}>
          {isScanning ? 'Đang quét...' : 'Quét BLE'}
        </button>
        {isScanning && (
          <button onClick={stopScan}>Dừng</button>
        )}
      </div>

      <div className="device-list">
        {devices.length === 0 && !isScanning && (
          <p>Chưa tìm thấy thiết bị nào</p>
        )}
        
        {devices.map(device => (
          <div key={device.deviceId} className="device-item">
            <h4>{device.name || 'Unnamed'}</h4>
            <p>ID: {device.deviceId}</p>
            <p>Signal: {device.RSSI} dBm</p>
            <button onClick={() => connectDevice(device.deviceId)}>
              Kết nối
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Filter by RSSI**: Chỉ hiển thị thiết bị có RSSI > -80 dBm
- **Debounce**: Tránh update UI quá nhanh
- **Stop khi không dùng**: Gọi `stopScan` để tiết kiệm pin
- **Service filter**: Dùng `services` để lọc thiết bị cần thiết
- **Permission check**: Kiểm tra quyền Bluetooth trước khi quét

### ❌ Không nên

- Quét liên tục không dừng (tốn pin)
- Hiển thị tất cả thiết bị (kể cả RSSI yếu)
- Không xử lý duplicate
- Quên check Bluetooth enabled

## Xử lý lỗi

```javascript
window.WindVane.call('WVBluetooth', 'scan', { interval: 10000 },
  (device) => { /* success */ },
  (error) => {
    switch(error.error) {
      case 'NO_PERMISSION':
        alert('Vui lòng cấp quyền Bluetooth');
        break;
      case 'BLUETOOTH_DISABLED':
        alert('Vui lòng bật Bluetooth');
        break;
      case 'LOCATION_DISABLED':
        // Android yêu cầu Location cho BLE scan
        alert('Vui lòng bật Location services');
        break;
      default:
        alert('Lỗi quét BLE: ' + error.errorMessage);
    }
  }
);
```

## Giới hạn

- Chỉ quét BLE, không hỗ trợ Bluetooth Classic
- Android yêu cầu Location permission
- Khoảng cách quét: ~10-50m (tùy môi trường)
- Battery impact cao nếu quét liên tục

## Common Service UUIDs

| Service | UUID |
|---------|------|
| Heart Rate | `0000180d-0000-1000-8000-00805f9b34fb` |
| Battery | `0000180f-0000-1000-8000-00805f9b34fb` |
| Device Info | `0000180a-0000-1000-8000-00805f9b34fb` |
| Environmental | `0000181a-0000-1000-8000-00805f9b34fb` |

## Xem thêm

- [Bluetooth: connect](./bluetooth_connect) - Kết nối thiết bị
- [Bluetooth: getServices](./bluetooth_getServices) - Lấy services
