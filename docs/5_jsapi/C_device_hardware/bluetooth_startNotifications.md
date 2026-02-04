---
sidebar_position: 17
---

# Bluetooth startNotifications

Bật notification cho characteristic để nhận thông báo khi giá trị thay đổi.

## API Call

```javascript
// Enable notifications
window.WindVane.call('WVBluetooth', 'startNotifications', {
  deviceId: 'XX:XX:XX:XX:XX:XX',
  serviceUUID: '0000180d-0000-1000-8000-00805f9b34fb',
  characteristicUUID: '00002a37-0000-1000-8000-00805f9b34fb'
});

// Listen for changes
window.WindVane.call('WVBluetooth', 'onCharacteristicValueChange', {}, (result) => {
  console.log('New value:', result.value);
});

// Stop notifications
window.WindVane.call('WVBluetooth', 'stopNotifications', {
  deviceId: 'XX:XX:XX:XX:XX:XX',
  serviceUUID: '0000180d-0000-1000-8000-00805f9b34fb',
  characteristicUUID: '00002a37-0000-1000-8000-00805f9b34fb'
});
```

## Use Case: Real-time Heart Rate

```javascript
class HeartRateMonitor {
  start(deviceId) {
    const serviceUUID = '0000180d-0000-1000-8000-00805f9b34fb';
    const charUUID = '00002a37-0000-1000-8000-00805f9b34fb';

    window.WindVane.call('WVBluetooth', 'startNotifications', {
      deviceId: deviceId,
      serviceUUID: serviceUUID,
      characteristicUUID: charUUID
    });

    window.WindVane.call('WVBluetooth', 'onCharacteristicValueChange', {}, (result) => {
      if (result.characteristicUUID === charUUID) {
        const bpm = new Uint8Array(result.value)[1];
        this.updateUI(bpm);
      }
    });
  }

  updateUI(bpm) {
    document.getElementById('heart-rate').textContent = `${bpm} BPM`;
    
    const color = bpm > 150 ? '#EE0033' : bpm > 100 ? '#FFA500' : '#00FF00';
    document.getElementById('heart-rate').style.color = color;
  }

  stop(deviceId) {
    window.WindVane.call('WVBluetooth', 'stopNotifications', {
      deviceId: deviceId,
      serviceUUID: '0000180d-0000-1000-8000-00805f9b34fb',
      characteristicUUID: '00002a37-0000-1000-8000-00805f9b34fb'
    });
  }
}
```
