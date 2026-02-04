---
sidebar_position: 2
---

# downloadFile - Tải file về thiết bị

Download file từ URL về local storage.

## API Call

```javascript
window.WindVane.call('WVFile', 'downloadFile', {
  url: 'https://example.com/file.pdf',
  fileName: 'document.pdf'
}, (result) => {
  console.log('Downloaded to:', result.filePath);
}, (error) => {
  console.error('Download failed:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `url` | `string` | Có | URL file cần tải |
| `fileName` | `string` | Không | Tên file lưu, mặc định từ URL |
| `header` | `object` | Không | HTTP headers |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `filePath` | `string` | Đường dẫn file đã tải |
| `fileSize` | `number` | Kích thước file (bytes) |

## Use Case: PDF Viewer

```javascript
class PDFDownloader {
  async downloadAndView(pdfUrl) {
    window.WindVane.call('WVUI', 'showLoadingBox', {
      text: 'Đang tải PDF...'
    });

    window.WindVane.call('WVFile', 'downloadFile', {
      url: pdfUrl,
      fileName: 'document.pdf'
    }, (result) => {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      
      // Mở PDF viewer
      window.WindVane.call('WVFile', 'openDocument', {
        filePath: result.filePath
      });
    }, (error) => {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      window.WindVane.call('WVUIDialog', 'alert', {
        message: 'Không thể tải file: ' + error.message
      });
    });
  }
}

// Sử dụng
const downloader = new PDFDownloader();
downloader.downloadAndView('https://example.com/invoice.pdf');
```

## Use Case: Download Manager với Progress

```javascript
class DownloadManager {
  downloads = new Map();

  startDownload(url, fileName) {
    const downloadId = Date.now() + Math.random();
    
    const downloadTask = window.WindVane.call('WVFile', 'downloadFile', {
      url: url,
      fileName: fileName
    }, (result) => {
      this.onComplete(downloadId, result);
    }, (error) => {
      this.onError(downloadId, error);
    });

    // Track progress
    downloadTask.onProgressUpdate((progress) => {
      const percent = Math.round(progress.totalBytesWritten / progress.totalBytesExpectedToWrite * 100);
      this.updateProgress(downloadId, percent);
    });

    this.downloads.set(downloadId, {
      task: downloadTask,
      url: url,
      fileName: fileName,
      progress: 0
    });

    return downloadId;
  }

  updateProgress(downloadId, percent) {
    const download = this.downloads.get(downloadId);
    if (download) {
      download.progress = percent;
      const el = document.getElementById(`download-${downloadId}`);
      if (el) {
        el.querySelector('.progress-bar').style.width = `${percent}%`;
        el.querySelector('.progress-text').textContent = `${percent}%`;
      }
    }
  }

  onComplete(downloadId, result) {
    const download = this.downloads.get(downloadId);
    console.log(`Download completed: ${download.fileName}`);
    
    window.WindVane.call('WVUIToast', 'toast', {
      message: `Đã tải xong ${download.fileName}`
    });
    
    this.downloads.delete(downloadId);
  }

  onError(downloadId, error) {
    console.error(`Download failed:`, error);
    this.downloads.delete(downloadId);
  }

  cancelDownload(downloadId) {
    const download = this.downloads.get(downloadId);
    if (download) {
      download.task.abort();
      this.downloads.delete(downloadId);
    }
  }
}

// Sử dụng
const manager = new DownloadManager();
const downloadId = manager.startDownload('https://example.com/video.mp4', 'video.mp4');

// Cancel nếu cần
document.getElementById('cancel-btn').onclick = () => {
  manager.cancelDownload(downloadId);
};
```

## Best Practices

:::warning Storage Permission
Đảm bảo có quyền ghi vào storage trước khi download.
:::

### 1. Check Available Space

```javascript
async function checkSpaceBeforeDownload(fileSize) {
  return new Promise((resolve) => {
    window.WindVane.call('WVSystem', 'getSystemInfo', {}, (result) => {
      const availableSpace = result.availableDiskSpace;
      
      if (availableSpace < fileSize * 1.2) { // Cần 20% buffer
        window.WindVane.call('WVUIDialog', 'alert', {
          message: 'Không đủ dung lượng để tải file'
        });
        resolve(false);
      } else {
        resolve(true);
      }
    });
  });
}
```

### 2. Network-aware Download

```javascript
async function smartDownload(url, fileSize) {
  return new Promise((resolve) => {
    window.WindVane.call('WVNetwork', 'getNetworkType', {}, (network) => {
      if (network.networkType !== 'wifi' && fileSize > 10 * 1024 * 1024) {
        window.WindVane.call('WVUIDialog', 'confirm', {
          message: `File ${(fileSize / 1024 / 1024).toFixed(1)}MB. Tiếp tục với ${network.networkType.toUpperCase()}?`,
          okbutton: 'Tiếp tục',
          cancelbutton: 'Hủy'
        }, () => startDownload(url), () => resolve(false));
      } else {
        startDownload(url);
      }
    });
  });
}
```

## API liên quan

- [uploadFile](./uploadFile) - Upload file lên server
- [chooseFiles](./chooseFiles) - Chọn file từ thiết bị
