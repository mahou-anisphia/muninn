---
sidebar_position: 22
---

# getSystemInfo - Thông tin hệ thống

Lấy thông tin chi tiết về thiết bị và hệ điều hành.

## API Call

```javascript
window.WindVane.call('WVSystem', 'getSystemInfo', {}, (result) => {
  console.log('Device:', result);
});
```

## Success Result

```javascript
{
  brand: 'Samsung',
  model: 'SM-G998B',
  system: 'Android',
  systemVersion: '13',
  platform: 'android',
  screenWidth: 1080,
  screenHeight: 2400,
  pixelRatio: 3,
  language: 'vi-VN',
  availableDiskSpace: 5000000000,
  totalDiskSpace: 128000000000,
  batteryLevel: 85,
  isCharging: false
}
```

## Use Case: Device Analytics

```javascript
class DeviceAnalytics {
  async collectDeviceInfo() {
    return new Promise((resolve) => {
      window.WindVane.call('WVSystem', 'getSystemInfo', {}, (info) => {
        const analytics = {
          device: `${info.brand} ${info.model}`,
          os: `${info.system} ${info.systemVersion}`,
          screen: `${info.screenWidth}x${info.screenHeight}`,
          language: info.language,
          storage: {
            available: (info.availableDiskSpace / 1024 / 1024 / 1024).toFixed(1) + ' GB',
            total: (info.totalDiskSpace / 1024 / 1024 / 1024).toFixed(0) + ' GB'
          }
        };
        
        resolve(analytics);
      });
    });
  }

  async sendToServer() {
    const deviceInfo = await this.collectDeviceInfo();
    
    fetch('https://api.example.com/analytics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(deviceInfo)
    });
  }
}

// Sử dụng
const analytics = new DeviceAnalytics();
await analytics.sendToServer();
```
