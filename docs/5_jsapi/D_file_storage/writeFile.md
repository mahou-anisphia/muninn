---
sidebar_position: 5
---

# writeFile - Ghi file local

Ghi dữ liệu vào file local.

## API Call

```javascript
window.WindVane.call('WVFile', 'write', {
  filePath: '/storage/emulated/0/Documents/log.txt',
  data: 'Log entry: user logged in',
  encoding: 'utf-8',
  mode: 'append'
}, (result) => {
  console.log('Write success');
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `filePath` | `string` | Có | Đường dẫn file |
| `data` | `string` | Có | Dữ liệu ghi |
| `encoding` | `string` | Không | `utf-8`, `base64` |
| `mode` | `string` | Không | `write` (ghi đè) hoặc `append` (nối thêm) |

## Use Case: Logger

```javascript
class FileLogger {
  constructor() {
    this.logFile = '/storage/emulated/0/Android/data/com.viettel.miniapp/files/app.log';
  }

  log(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [${level}] ${message}\n`;
    
    window.WindVane.call('WVFile', 'write', {
      filePath: this.logFile,
      data: logEntry,
      mode: 'append'
    });
  }

  info(message) {
    this.log('INFO', message);
  }

  error(message) {
    this.log('ERROR', message);
  }

  async readLogs() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'read', {
        filePath: this.logFile
      }, (result) => resolve(result.data), reject);
    });
  }
}

// Sử dụng
const logger = new FileLogger();
logger.info('App started');
logger.error('Failed to load data');

// Đọc logs
const logs = await logger.readLogs();
console.log(logs);
```
