---
sidebar_position: 4
---

# readFile - Đọc file local

Đọc nội dung file từ local storage.

## API Call

```javascript
window.WindVane.call('WVFile', 'read', {
  filePath: '/storage/emulated/0/Documents/config.json',
  encoding: 'utf-8'
}, (result) => {
  console.log('File content:', result.data);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `filePath` | `string` | Có | Đường dẫn file |
| `encoding` | `string` | Không | Encoding: `utf-8`, `base64` |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `data` | `string` | Nội dung file |

## Use Case: Config Reader

```javascript
class ConfigManager {
  async loadConfig() {
    return new Promise((resolve, reject) => {
      const configPath = `${this.getAppDir()}/config.json`;
      
      window.WindVane.call('WVFile', 'read', {
        filePath: configPath,
        encoding: 'utf-8'
      }, (result) => {
        try {
          const config = JSON.parse(result.data);
          resolve(config);
        } catch (e) {
          reject(new Error('Invalid config file'));
        }
      }, reject);
    });
  }

  getAppDir() {
    // Get app's private directory
    return '/storage/emulated/0/Android/data/com.viettel.miniapp/files';
  }
}

// Sử dụng
const configMgr = new ConfigManager();
const config = await configMgr.loadConfig();
console.log('Settings:', config);
```
