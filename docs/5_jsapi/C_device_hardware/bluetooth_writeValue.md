---
sidebar_position: 16
---

# Bluetooth writeValue - Ghi characteristic

Ghi dữ liệu vào BLE characteristic.

## API Call

```javascript
window.WindVane.call('WVBluetooth', 'writeValue', {
  deviceId: 'XX:XX:XX:XX:XX:XX',
  serviceUUID: '0000ffe0-0000-1000-8000-00805f9b34fb',
  characteristicUUID: '0000ffe1-0000-1000-8000-00805f9b34fb',
  value: new Uint8Array([0x01, 0x02, 0x03])
}, (result) => {
  console.log('Write successful');
}, (error) => {
  console.error('Write failed:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `deviceId` | `string` | Có | MAC address |
| `serviceUUID` | `string` | Có | UUID service |
| `characteristicUUID` | `string` | Có | UUID characteristic |
| `value` | `ArrayBuffer` \| `Uint8Array` | Có | Dữ liệu ghi (binary) |

## Use Case: LED Controller

```javascript
class BLELEDController {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.serviceUUID = '0000ffe0-0000-1000-8000-00805f9b34fb';
    this.charUUID = '0000ffe1-0000-1000-8000-00805f9b34fb';
  }

  async setColor(r, g, b) {
    const command = new Uint8Array([0x01, r, g, b]);
    
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'writeValue', {
        deviceId: this.deviceId,
        serviceUUID: this.serviceUUID,
        characteristicUUID: this.charUUID,
        value: command
      }, resolve, reject);
    });
  }

  async setBrightness(level) {
    const command = new Uint8Array([0x02, Math.min(255, Math.max(0, level))]);
    
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'writeValue', {
        deviceId: this.deviceId,
        serviceUUID: this.serviceUUID,
        characteristicUUID: this.charUUID,
        value: command
      }, resolve, reject);
    });
  }

  async turnOff() {
    const command = new Uint8Array([0x00]);
    
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'writeValue', {
        deviceId: this.deviceId,
        serviceUUID: this.serviceUUID,
        characteristicUUID: this.charUUID,
        value: command
      }, resolve, reject);
    });
  }
}

// Sử dụng
const led = new BLELEDController('AA:BB:CC:DD:EE:FF');

// Đỏ
await led.setColor(255, 0, 0);

// Xanh lá 50% brightness
await led.setColor(0, 255, 0);
await led.setBrightness(128);

// Tắt
await led.turnOff();
```

## API liên quan

- [bluetooth_readValue](./bluetooth_readValue) - Đọc dữ liệu
