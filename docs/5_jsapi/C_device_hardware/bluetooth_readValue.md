---
sidebar_position: 15
---

# Bluetooth readValue - Đọc characteristic

Đọc giá trị từ một BLE characteristic.

## API Call

```javascript
window.WindVane.call('WVBluetooth', 'readValue', {
  deviceId: 'XX:XX:XX:XX:XX:XX',
  serviceUUID: '0000180a-0000-1000-8000-00805f9b34fb',
  characteristicUUID: '00002a29-0000-1000-8000-00805f9b34fb'
}, (result) => {
  console.log('Value:', result.value);
}, (error) => {
  console.error('Read failed:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `deviceId` | `string` | Có | MAC address thiết bị |
| `serviceUUID` | `string` | Có | UUID của service |
| `characteristicUUID` | `string` | Có | UUID của characteristic |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `value` | `ArrayBuffer` | Dữ liệu đọc được (binary) |

## Use Case: Device Info Reader

```javascript
class DeviceInfoReader {
  constructor(deviceId) {
    this.deviceId = deviceId;
    this.serviceUUID = '0000180a-0000-1000-8000-00805f9b34fb';
    this.characteristics = {
      manufacturer: '00002a29-0000-1000-8000-00805f9b34fb',
      model: '00002a24-0000-1000-8000-00805f9b34fb',
      serial: '00002a25-0000-1000-8000-00805f9b34fb',
      firmware: '00002a26-0000-1000-8000-00805f9b34fb'
    };
  }

  async readString(charUUID) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVBluetooth', 'readValue', {
        deviceId: this.deviceId,
        serviceUUID: this.serviceUUID,
        characteristicUUID: charUUID
      }, (result) => {
        const decoder = new TextDecoder('utf-8');
        const text = decoder.decode(result.value);
        resolve(text);
      }, reject);
    });
  }

  async getAllInfo() {
    try {
      const info = {
        manufacturer: await this.readString(this.characteristics.manufacturer),
        model: await this.readString(this.characteristics.model),
        serial: await this.readString(this.characteristics.serial),
        firmware: await this.readString(this.characteristics.firmware)
      };
      return info;
    } catch (error) {
      console.error('Failed to read device info:', error);
      return null;
    }
  }

  displayInfo() {
    this.getAllInfo().then(info => {
      if (info) {
        document.getElementById('device-info').innerHTML = `
          <div class="info-card">
            <h3>Thông tin thiết bị</h3>
            <p><strong>Hãng:</strong> ${info.manufacturer}</p>
            <p><strong>Model:</strong> ${info.model}</p>
            <p><strong>Serial:</strong> ${info.serial}</p>
            <p><strong>Firmware:</strong> ${info.firmware}</p>
          </div>
        `;
      }
    });
  }
}

// Sử dụng
const reader = new DeviceInfoReader('AA:BB:CC:DD:EE:FF');
await reader.displayInfo();
```

## Best Practices

:::warning Data Parsing
ArrayBuffer cần parse đúng format. Sử dụng DataView hoặc TypedArray.
:::

```javascript
// Parse số nguyên 16-bit
const value = new DataView(result.value).getUint16(0, true);

// Parse string
const text = new TextDecoder().decode(result.value);
```

## API liên quan

- [bluetooth_writeValue](./bluetooth_writeValue) - Ghi dữ liệu
- [bluetooth_startNotifications](./bluetooth_startNotifications) - Nhận thông báo thay đổi
