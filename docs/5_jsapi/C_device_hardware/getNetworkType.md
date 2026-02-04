---
sidebar_position: 20
---

# getNetworkType - Loại mạng đang kết nối

Lấy thông tin về loại mạng hiện tại (Wifi, 4G, 5G...).

## API Call

```javascript
window.WindVane.call('WVNetwork', 'getNetworkType', {}, (result) => {
  console.log('Network type:', result.networkType);
});
```

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `networkType` | `string` | Loại mạng: `wifi`, `2g`, `3g`, `4g`, `5g`, `none` |
| `isConnected` | `boolean` | Có kết nối internet không |

## Use Case: Adaptive Video Quality

```javascript
class AdaptiveVideoPlayer {
  async selectQuality() {
    return new Promise((resolve) => {
      window.WindVane.call('WVNetwork', 'getNetworkType', {}, (result) => {
        let quality;
        
        switch (result.networkType) {
          case 'wifi':
          case '5g':
            quality = '1080p';
            break;
          case '4g':
            quality = '720p';
            break;
          case '3g':
            quality = '480p';
            break;
          default:
            quality = '360p';
        }
        
        resolve(quality);
      });
    });
  }

  async playVideo(videoUrl) {
    const quality = await this.selectQuality();
    const videoSource = `${videoUrl}?quality=${quality}`;
    
    document.querySelector('video').src = videoSource;
  }
}

// Sử dụng
const player = new AdaptiveVideoPlayer();
await player.playVideo('https://example.com/video.mp4');
```

## Use Case: Download Manager

```javascript
class DownloadManager {
  async checkNetworkBeforeDownload(fileSize) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVNetwork', 'getNetworkType', {}, (result) => {
        if (!result.isConnected) {
          reject(new Error('Không có kết nối internet'));
          return;
        }

        // Cảnh báo nếu dùng mobile data cho file lớn
        if (result.networkType !== 'wifi' && fileSize > 50 * 1024 * 1024) { // 50MB
          window.WindVane.call('WVUIDialog', 'confirm', {
            title: 'Cảnh báo',
            message: `File ${(fileSize / 1024 / 1024).toFixed(1)}MB. Tiếp tục tải với ${result.networkType.toUpperCase()}?`,
            okbutton: 'Tiếp tục',
            cancelbutton: 'Hủy'
          }, () => resolve(true), () => resolve(false));
        } else {
          resolve(true);
        }
      });
    });
  }

  async downloadFile(url, fileSize) {
    const canDownload = await this.checkNetworkBeforeDownload(fileSize);
    
    if (!canDownload) {
      window.WindVane.call('WVUIToast', 'toast', {
        message: 'Đã hủy tải xuống'
      });
      return;
    }

    window.WindVane.call('WVFile', 'downloadFile', {
      url: url
    }, (result) => {
      window.WindVane.call('WVUIToast', 'toast', {
        message: 'Tải xuống thành công'
      });
    });
  }
}

// Sử dụng
const dm = new DownloadManager();
await dm.downloadFile('https://example.com/large-file.zip', 100 * 1024 * 1024); // 100MB
```

## Best Practices

:::tip Network Monitoring
Lắng nghe thay đổi mạng để adapt realtime.
:::

```javascript
// Monitor network changes
window.WindVane.call('WVNetwork', 'onNetworkStatusChange', {}, (result) => {
  console.log('Network changed to:', result.networkType);
  handleNetworkChange(result);
});
```

## API liên quan

- [getSystemInfo](./getSystemInfo) - Thông tin hệ thống
