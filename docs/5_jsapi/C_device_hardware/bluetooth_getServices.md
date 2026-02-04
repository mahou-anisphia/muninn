---
sidebar_position: 13
---

# Bluetooth getServices - Lấy danh sách services

Lấy danh sách GATT services từ thiết bị BLE đã kết nối.

## API Call

```javascript
window.WindVane.call('WVBluetooth', 'getServices', {
  deviceId: 'XX:XX:XX:XX:XX:XX'
}, (result) => {
  console.log('Services:', result.services);
}, (error) => {
  console.error('Failed to get services:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `deviceId` | `string` | Có | MAC address của thiết bị đã kết nối |

## Success Result

```javascript
{
  services: [
    {
      uuid: '0000180a-0000-1000-8000-00805f9b34fb',
      isPrimary: true
    },
    {
      uuid: '0000180f-0000-1000-8000-00805f9b34fb',
      isPrimary: true
    }
  ]
}
```

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `services` | `array` | Danh sách GATT services |
| `services[].uuid` | `string` | UUID của service |
| `services[].isPrimary` | `boolean` | Service chính hay phụ |

## Use Case 1: Service Discovery với Known UUIDs

```javascript
class BLEServiceDiscovery {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.knownServices = {
      '0000180a-0000-1000-8000-00805f9b34fb': 'Device Information',
      '0000180f-0000-1000-8000-00805f9b34fb': 'Battery Service',
      '0000180d-0000-1000-8000-00805f9b34fb': 'Heart Rate',
      '00001812-0000-1000-8000-00805f9b34fb': 'HID Service'
    };
  }

  async discoverServices() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'getServices', {
        deviceId: this.deviceId
      }, (result) => {
        const services = result.services.map(service => ({
          uuid: service.uuid,
          name: this.knownServices[service.uuid] || 'Unknown Service',
          isPrimary: service.isPrimary
        }));
        
        resolve(services);
      }, reject);
    });
  }

  async displayServices() {
    try {
      const services = await this.discoverServices();
      
      const container = document.getElementById('services-list');
      container.innerHTML = services.map(s => `
        <div class="service-item ${s.isPrimary ? 'primary' : 'secondary'}">
          <div class="service-name">${s.name}</div>
          <div class="service-uuid">${s.uuid}</div>
          <button onclick="exploreService('${s.uuid}')">Xem chi tiết</button>
        </div>
      `).join('');
      
    } catch (error) {
      console.error('Service discovery failed:', error);
    }
  }
}

// Sử dụng
const discovery = new BLEServiceDiscovery('AA:BB:CC:DD:EE:FF');
await discovery.displayServices();
```

## Use Case 2: Heart Rate Monitor

```javascript
class HeartRateMonitor {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.heartRateServiceUUID = '0000180d-0000-1000-8000-00805f9b34fb';
    this.heartRateCharUUID = '00002a37-0000-1000-8000-00805f9b34fb';
  }

  async init() {
    // Bước 1: Lấy danh sách services
    const services = await this.getServices();
    
    // Bước 2: Kiểm tra Heart Rate service có tồn tại không
    const hasHeartRate = services.some(s => 
      s.uuid.toLowerCase() === this.heartRateServiceUUID.toLowerCase()
    );
    
    if (!hasHeartRate) {
      throw new Error('Thiết bị không hỗ trợ Heart Rate');
    }
    
    // Bước 3: Lấy characteristics
    const characteristics = await this.getCharacteristics();
    
    // Bước 4: Bật notification
    await this.enableNotifications();
    
    return true;
  }

  getServices() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'getServices', {
        deviceId: this.deviceId
      }, (result) => resolve(result.services), reject);
    });
  }

  getCharacteristics() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'getCharacteristics', {
        deviceId: this.deviceId,
        serviceUUID: this.heartRateServiceUUID
      }, (result) => resolve(result.characteristics), reject);
    });
  }

  enableNotifications() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'startNotifications', {
        deviceId: this.deviceId,
        serviceUUID: this.heartRateServiceUUID,
        characteristicUUID: this.heartRateCharUUID
      }, resolve, reject);
    });
  }

  onHeartRateChange(callback) {
    window.WindVane.call('WVBluetooth', 'onCharacteristicValueChange', {}, (result) => {
      if (result.characteristicUUID === this.heartRateCharUUID) {
        // Parse Heart Rate Measurement
        const dataView = new DataView(result.value.buffer);
        const flags = dataView.getUint8(0);
        const rate = (flags & 0x01) ? dataView.getUint16(1, true) : dataView.getUint8(1);
        
        callback(rate);
      }
    });
  }
}

// Sử dụng
const hrm = new HeartRateMonitor('11:22:33:44:55:66');

try {
  await hrm.init();
  
  hrm.onHeartRateChange((bpm) => {
    document.getElementById('heart-rate').textContent = `${bpm} BPM`;
    
    // Cảnh báo nếu nhịp tim bất thường
    if (bpm > 180 || bpm < 40) {
      window.WindVane.call('WVUIDialog', 'alert', {
        message: `Nhịp tim bất thường: ${bpm} BPM`
      });
    }
  });
  
} catch (error) {
  alert('Không thể khởi động Heart Rate Monitor: ' + error.message);
}
```

## Use Case 3: Battery Service Reader

```javascript
class BatteryServiceReader {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.batteryServiceUUID = '0000180f-0000-1000-8000-00805f9b34fb';
    this.batteryLevelCharUUID = '00002a19-0000-1000-8000-00805f9b34fb';
  }

  async checkBatterySupport() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'getServices', {
        deviceId: this.deviceId
      }, (result) => {
        const hasBattery = result.services.some(s => 
          s.uuid.toLowerCase() === this.batteryServiceUUID.toLowerCase()
        );
        resolve(hasBattery);
      }, reject);
    });
  }

  async readBatteryLevel() {
    const hasSupport = await this.checkBatterySupport();
    if (!hasSupport) {
      throw new Error('Thiết bị không hỗ trợ Battery Service');
    }

    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'readValue', {
        deviceId: this.deviceId,
        serviceUUID: this.batteryServiceUUID,
        characteristicUUID: this.batteryLevelCharUUID
      }, (result) => {
        // Battery level là 1 byte (0-100%)
        const level = new Uint8Array(result.value)[0];
        resolve(level);
      }, reject);
    });
  }

  async monitorBattery(callback, intervalMs = 60000) {
    const interval = setInterval(async () => {
      try {
        const level = await this.readBatteryLevel();
        callback(level);
        
        // Cảnh báo pin yếu
        if (level < 20) {
          window.WindVane.call('WVUIToast', 'toast', {
            message: `Pin thiết bị còn ${level}%`
          });
        }
      } catch (error) {
        console.error('Failed to read battery:', error);
      }
    }, intervalMs);

    return () => clearInterval(interval);
  }
}

// Sử dụng
const battery = new BatteryServiceReader('AA:BB:CC:DD:EE:FF');

// Đọc 1 lần
const level = await battery.readBatteryLevel();
console.log(`Battery: ${level}%`);

// Monitor liên tục
const stopMonitoring = await battery.monitorBattery((level) => {
  document.getElementById('device-battery').textContent = `${level}%`;
}, 30000); // Check mỗi 30 giây

// Cleanup
window.addEventListener('beforeunload', stopMonitoring);
```

## Best Practices

:::info Service UUIDs
Sử dụng GATT standard UUIDs khi có thể. Tham khảo: https://www.bluetooth.com/specifications/gatt/
:::

### 1. Cache Services

```javascript
class ServiceCache {
  constructor() {
    this.cache = new Map();
  }

  async getServices(deviceId, forceRefresh = false) {
    if (!forceRefresh && this.cache.has(deviceId)) {
      return this.cache.get(deviceId);
    }

    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'getServices', {
        deviceId: deviceId
      }, (result) => {
        this.cache.set(deviceId, result.services);
        resolve(result.services);
      }, reject);
    });
  }

  clearCache(deviceId) {
    if (deviceId) {
      this.cache.delete(deviceId);
    } else {
      this.cache.clear();
    }
  }
}

const cache = new ServiceCache();

// Lần đầu: gọi API
const services = await cache.getServices('AA:BB:CC:DD:EE:FF');

// Lần sau: lấy từ cache
const cachedServices = await cache.getServices('AA:BB:CC:DD:EE:FF');

// Force refresh khi cần
const freshServices = await cache.getServices('AA:BB:CC:DD:EE:FF', true);
```

### 2. Filter Primary Services Only

```javascript
async function getPrimaryServices(deviceId) {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVBluetooth', 'getServices', {
      deviceId: deviceId
    }, (result) => {
      const primaryServices = result.services.filter(s => s.isPrimary);
      resolve(primaryServices);
    }, reject);
  });
}
```

### 3. Service Name Resolver

```javascript
const GATT_SERVICES = {
  '00001800-0000-1000-8000-00805f9b34fb': 'Generic Access',
  '00001801-0000-1000-8000-00805f9b34fb': 'Generic Attribute',
  '0000180a-0000-1000-8000-00805f9b34fb': 'Device Information',
  '0000180f-0000-1000-8000-00805f9b34fb': 'Battery Service',
  '0000180d-0000-1000-8000-00805f9b34fb': 'Heart Rate',
  '00001816-0000-1000-8000-00805f9b34fb': 'Cycling Speed and Cadence',
  '00001818-0000-1000-8000-00805f9b34fb': 'Cycling Power'
};

function resolveServiceName(uuid) {
  return GATT_SERVICES[uuid.toLowerCase()] || `Custom Service (${uuid})`;
}
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `NOT_CONNECTED` | Chưa kết nối thiết bị | Gọi connect() trước |
| `SERVICE_DISCOVERY_FAILED` | Thiết bị không phản hồi | Reconnect và thử lại |
| `TIMEOUT` | Discovery mất quá lâu | Kiểm tra khoảng cách, tăng timeout |

## API liên quan

- [bluetooth_connect](./bluetooth_connect) - Kết nối trước khi get services
- [bluetooth_getCharacteristics](./bluetooth_getCharacteristics) - Lấy characteristics từ service
- [bluetooth_readValue](./bluetooth_readValue) - Đọc dữ liệu từ characteristic
