---
sidebar_position: 14
---

# Bluetooth getCharacteristics

Lấy danh sách characteristics từ một service.

## API Call

```javascript
window.WindVane.call('WVBluetooth', 'getCharacteristics', {
  deviceId: 'XX:XX:XX:XX:XX:XX',
  serviceUUID: '0000180a-0000-1000-8000-00805f9b34fb'
}, (result) => {
  console.log('Characteristics:', result.characteristics);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `deviceId` | `string` | Có | MAC address |
| `serviceUUID` | `string` | Có | UUID của service |

## Success Result

```javascript
{
  characteristics: [
    {
      uuid: '00002a29-0000-1000-8000-00805f9b34fb',
      properties: {
        read: true,
        write: false,
        notify: false
      }
    }
  ]
}
```

## Use Case: Explore Service

```javascript
async function exploreService(deviceId, serviceUUID) {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVBluetooth', 'getCharacteristics', {
      deviceId: deviceId,
      serviceUUID: serviceUUID
    }, (result) => {
      const chars = result.characteristics.map(c => ({
        uuid: c.uuid,
        readable: c.properties.read,
        writable: c.properties.write,
        notifiable: c.properties.notify
      }));
      resolve(chars);
    }, reject);
  });
}

// Sử dụng
const chars = await exploreService('AA:BB:CC:DD:EE:FF', '0000180a-0000-1000-8000-00805f9b34fb');
console.log('Available characteristics:', chars);
```
