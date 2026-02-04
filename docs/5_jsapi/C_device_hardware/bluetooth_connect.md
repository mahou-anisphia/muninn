---
sidebar_position: 12
---

# Bluetooth Connect - Kết nối thiết bị BLE

Kết nối đến thiết bị Bluetooth Low Energy đã quét được.

## API Call

```javascript
window.WindVane.call('WVBluetooth', 'connect', {
  deviceId: 'XX:XX:XX:XX:XX:XX'
}, (result) => {
  console.log('Connected successfully');
}, (error) => {
  console.error('Connection failed:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `deviceId` | `string` | Có | MAC address của thiết bị BLE |
| `timeout` | `number` | Không | Timeout kết nối (ms), mặc định 10000 |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `deviceId` | `string` | Device ID đã kết nối |
| `connected` | `boolean` | Trạng thái kết nối |

## Use Case 1: Smart Lock

```javascript
class SmartLockController {
  constructor() {
    this.deviceId = null;
    this.isConnected = false;
  }

  async connect(deviceId) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'connect', {
        deviceId: deviceId,
        timeout: 15000 // 15 giây cho smart lock
      }, (result) => {
        this.deviceId = deviceId;
        this.isConnected = true;
        console.log('Connected to smart lock');
        resolve(result);
      }, (error) => {
        reject(error);
      });
    });
  }

  async unlock() {
    if (!this.isConnected) {
      throw new Error('Not connected to lock');
    }

    // Gửi lệnh unlock qua characteristic
    const serviceUUID = '0000180a-0000-1000-8000-00805f9b34fb';
    const charUUID = '00002a29-0000-1000-8000-00805f9b34fb';
    
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'writeValue', {
        deviceId: this.deviceId,
        serviceUUID: serviceUUID,
        characteristicUUID: charUUID,
        value: 'UNLOCK' // Command byte array
      }, (result) => {
        console.log('Lock unlocked');
        resolve(result);
      }, reject);
    });
  }

  disconnect() {
    if (this.isConnected) {
      window.WindVane.call('WVBluetooth', 'disconnect', {
        deviceId: this.deviceId
      });
      this.isConnected = false;
    }
  }
}

// Sử dụng
const lock = new SmartLockController();

// Sau khi scan và tìm thấy lock
const deviceId = 'AA:BB:CC:DD:EE:FF';
try {
  await lock.connect(deviceId);
  await lock.unlock();
  
  window.WindVane.call('WVUIToast', 'toast', {
    message: 'Đã mở khóa thành công!'
  });
  
  // Tự động ngắt kết nối sau 5 giây
  setTimeout(() => lock.disconnect(), 5000);
} catch (error) {
  window.WindVane.call('WVUIToast', 'toast', {
    message: 'Không thể kết nối khóa: ' + error.message
  });
}
```

## Use Case 2: Fitness Tracker với Retry

```javascript
class FitnessTrackerConnection {
  constructor() {
    this.maxRetries = 3;
    this.retryDelay = 2000;
  }

  async connectWithRetry(deviceId) {
    let lastError;
    
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        console.log(`Connection attempt ${attempt}/${this.maxRetries}`);
        
        const result = await this.connect(deviceId);
        console.log('Connected successfully');
        return result;
        
      } catch (error) {
        lastError = error;
        console.error(`Attempt ${attempt} failed:`, error);
        
        if (attempt < this.maxRetries) {
          await this.delay(this.retryDelay);
        }
      }
    }
    
    throw new Error(`Failed to connect after ${this.maxRetries} attempts: ${lastError.message}`);
  }

  connect(deviceId) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'connect', {
        deviceId: deviceId,
        timeout: 10000
      }, resolve, reject);
    });
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Sử dụng
const tracker = new FitnessTrackerConnection();

try {
  await tracker.connectWithRetry('11:22:33:44:55:66');
  // Bắt đầu đọc heart rate, steps...
} catch (error) {
  alert('Không thể kết nối vòng tay: ' + error.message);
}
```

## Use Case 3: Multi-device Connection Manager

```javascript
class BluetoothConnectionManager {
  constructor() {
    this.connections = new Map();
    this.connectionCallbacks = new Map();
  }

  async connect(deviceId, name) {
    if (this.connections.has(deviceId)) {
      console.log(`Already connected to ${name}`);
      return this.connections.get(deviceId);
    }

    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'connect', {
        deviceId: deviceId
      }, (result) => {
        const device = {
          id: deviceId,
          name: name,
          connected: true,
          connectedAt: Date.now()
        };
        
        this.connections.set(deviceId, device);
        this.notifyListeners('connected', device);
        
        resolve(device);
      }, (error) => {
        this.notifyListeners('error', { deviceId, error });
        reject(error);
      });
    });
  }

  disconnect(deviceId) {
    const device = this.connections.get(deviceId);
    if (!device) return;

    window.WindVane.call('WVBluetooth', 'disconnect', {
      deviceId: deviceId
    });

    this.connections.delete(deviceId);
    this.notifyListeners('disconnected', device);
  }

  disconnectAll() {
    this.connections.forEach((device, deviceId) => {
      this.disconnect(deviceId);
    });
  }

  getConnectedDevices() {
    return Array.from(this.connections.values());
  }

  isConnected(deviceId) {
    return this.connections.has(deviceId);
  }

  onConnectionChange(callback) {
    const id = Date.now() + Math.random();
    this.connectionCallbacks.set(id, callback);
    return () => this.connectionCallbacks.delete(id);
  }

  notifyListeners(event, data) {
    this.connectionCallbacks.forEach(callback => {
      callback(event, data);
    });
  }
}

// Sử dụng
const manager = new BluetoothConnectionManager();

// Lắng nghe thay đổi kết nối
const unsubscribe = manager.onConnectionChange((event, data) => {
  console.log(`Event: ${event}`, data);
  updateUIList();
});

// Kết nối nhiều thiết bị
await manager.connect('AA:BB:CC:DD:EE:FF', 'Heart Rate Monitor');
await manager.connect('11:22:33:44:55:66', 'Smart Watch');
await manager.connect('99:88:77:66:55:44', 'Bike Sensor');

// Hiển thị danh sách
function updateUIList() {
  const devices = manager.getConnectedDevices();
  document.getElementById('device-list').innerHTML = devices.map(d => `
    <div class="device-item">
      <span>${d.name}</span>
      <button onclick="manager.disconnect('${d.id}')">Ngắt</button>
    </div>
  `).join('');
}

// Cleanup khi thoát app
window.addEventListener('beforeunload', () => {
  manager.disconnectAll();
  unsubscribe();
});
```

## Best Practices

:::warning Connection Lifecycle
Luôn disconnect khi không sử dụng để tiết kiệm pin và giải phóng tài nguyên.
:::

### 1. Kiểm tra trước khi kết nối

```javascript
async function safeConnect(deviceId) {
  // Kiểm tra Bluetooth đã bật chưa
  window.WindVane.call('WVBluetooth', 'getAdapterState', {}, (state) => {
    if (!state.enabled) {
      window.WindVane.call('WVUIDialog', 'alert', {
        message: 'Vui lòng bật Bluetooth'
      });
      return;
    }
    
    // Tiến hành kết nối
    window.WindVane.call('WVBluetooth', 'connect', {
      deviceId: deviceId
    }, onConnected, onError);
  });
}
```

### 2. Timeout và Error Handling

```javascript
function connectWithCustomTimeout(deviceId, timeoutMs = 8000) {
  return new Promise((resolve, reject) => {
    let timeoutId;
    let completed = false;

    const complete = (success, data) => {
      if (completed) return;
      completed = true;
      clearTimeout(timeoutId);
      success ? resolve(data) : reject(data);
    };

    // Set timeout
    timeoutId = setTimeout(() => {
      complete(false, new Error('Connection timeout'));
    }, timeoutMs);

    // Attempt connection
    window.WindVane.call('WVBluetooth', 'connect', {
      deviceId: deviceId
    }, 
    (result) => complete(true, result),
    (error) => complete(false, error)
    );
  });
}
```

### 3. Connection Status Monitoring

```javascript
class ConnectionMonitor {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.checkInterval = null;
  }

  startMonitoring() {
    this.checkInterval = setInterval(() => {
      window.WindVane.call('WVBluetooth', 'getConnectedDevices', {}, (result) => {
        const isConnected = result.devices.some(d => d.id === this.deviceId);
        
        if (!isConnected) {
          console.warn('Device disconnected unexpectedly');
          this.handleDisconnection();
        }
      });
    }, 5000); // Check mỗi 5 giây
  }

  handleDisconnection() {
    this.stopMonitoring();
    
    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Mất kết nối thiết bị. Đang thử kết nối lại...'
    });
    
    // Attempt reconnect
    setTimeout(() => {
      window.WindVane.call('WVBluetooth', 'connect', {
        deviceId: this.deviceId
      }, () => {
        this.startMonitoring();
      });
    }, 2000);
  }

  stopMonitoring() {
    if (this.checkInterval) {
      clearInterval(this.checkInterval);
      this.checkInterval = null;
    }
  }
}
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `TIMEOUT` | Thiết bị không phản hồi | Tăng timeout, kiểm tra khoảng cách |
| `ALREADY_CONNECTED` | Đã kết nối rồi | Disconnect trước khi connect lại |
| `DEVICE_NOT_FOUND` | Device ID không tồn tại | Scan lại để lấy ID mới |
| `BLUETOOTH_DISABLED` | Bluetooth chưa bật | Request user bật Bluetooth |

## API liên quan

- [bluetooth_scan](./bluetooth_scan) - Quét thiết bị trước khi connect
- [bluetooth_getServices](./bluetooth_getServices) - Lấy services sau khi connect
- [bluetooth_writeValue](./bluetooth_writeValue) - Ghi dữ liệu sau khi connect
