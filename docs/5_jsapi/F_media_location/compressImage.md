---
sidebar_position: 5
---

# compressImage - Nén ảnh

Nén ảnh để giảm dung lượng file.

## API Call

```javascript
window.WindVane.call('WVImage', 'compressImage', {
  src: '/storage/emulated/0/DCIM/photo.jpg',
  quality: 0.7,
  maxWidth: 1920,
  maxHeight: 1080
}, (result) => {
  console.log('Compressed image:', result.path);
  console.log('Original size:', result.originalSize);
  console.log('Compressed size:', result.compressedSize);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `src` | `string` | Có | Đường dẫn ảnh gốc |
| `quality` | `number` | Không | Chất lượng 0-1, mặc định 0.8 |
| `maxWidth` | `number` | Không | Chiều rộng tối đa (px) |
| `maxHeight` | `number` | Không | Chiều cao tối đa (px) |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `path` | `string` | Đường dẫn ảnh đã nén |
| `originalSize` | `number` | Dung lượng gốc (bytes) |
| `compressedSize` | `number` | Dung lượng sau nén (bytes) |
| `width` | `number` | Chiều rộng ảnh mới |
| `height` | `number` | Chiều cao ảnh mới |

## Use Case 1: Smart Image Upload

```javascript
class SmartImageUploader {
  constructor() {
    this.maxFileSize = 2 * 1024 * 1024; // 2MB
  }

  async selectAndUpload() {
    window.WindVane.call('WVCamera', 'takePhoto', {
      mode: 'both'
    }, async (photo) => {
      let imagePath = photo.url;
      
      // Kiểm tra kích thước
      const fileInfo = await this.getFileInfo(imagePath);
      
      if (fileInfo.size > this.maxFileSize) {
        console.log(`File ${(fileInfo.size / 1024 / 1024).toFixed(1)}MB, compressing...`);
        imagePath = await this.compress(imagePath);
      }
      
      await this.upload(imagePath);
    });
  }

  async compress(imagePath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVImage', 'compressImage', {
        src: imagePath,
        quality: 0.7,
        maxWidth: 1920,
        maxHeight: 1080
      }, (result) => {
        const savedBytes = result.originalSize - result.compressedSize;
        const savedPercent = (savedBytes / result.originalSize * 100).toFixed(0);
        
        console.log(`Compressed: ${(savedBytes / 1024).toFixed(0)}KB saved (${savedPercent}%)`);
        
        resolve(result.path);
      }, reject);
    });
  }

  async getFileInfo(path) {
    return new Promise((resolve) => {
      window.WindVane.call('WVFile', 'getFileInfo', {
        filePath: path
      }, resolve);
    });
  }

  async upload(imagePath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'uploadFile', {
        url: 'https://api.example.com/upload',
        filePath: imagePath,
        name: 'image'
      }, resolve, reject);
    });
  }
}

// Sử dụng
const uploader = new SmartImageUploader();
uploader.selectAndUpload();
```

## Use Case 2: Batch Compress với Progress

```javascript
class BatchImageCompressor {
  constructor() {
    this.queue = [];
    this.processing = false;
  }

  async addToQueue(imagePaths) {
    this.queue.push(...imagePaths);
    
    if (!this.processing) {
      await this.processQueue();
    }
  }

  async processQueue() {
    this.processing = true;
    const totalImages = this.queue.length;
    let processed = 0;

    while (this.queue.length > 0) {
      const imagePath = this.queue.shift();
      
      try {
        await this.compressImage(imagePath);
        processed++;
        
        // Update progress
        const percent = Math.round(processed / totalImages * 100);
        this.updateProgress(percent, processed, totalImages);
        
      } catch (error) {
        console.error(`Failed to compress ${imagePath}:`, error);
      }
    }

    this.processing = false;
    this.onComplete(processed, totalImages);
  }

  async compressImage(imagePath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVImage', 'compressImage', {
        src: imagePath,
        quality: 0.6,
        maxWidth: 1280,
        maxHeight: 720
      }, resolve, reject);
    });
  }

  updateProgress(percent, current, total) {
    const progressBar = document.getElementById('compress-progress');
    const progressText = document.getElementById('compress-text');
    
    if (progressBar) {
      progressBar.style.width = `${percent}%`;
    }
    
    if (progressText) {
      progressText.textContent = `Đã nén ${current}/${total} ảnh (${percent}%)`;
    }
  }

  onComplete(processed, total) {
    window.WindVane.call('WVUIToast', 'toast', {
      message: `Đã nén ${processed}/${total} ảnh thành công`
    });
  }
}

// Sử dụng
const compressor = new BatchImageCompressor();

// Select multiple images
window.WindVane.call('WVCamera', 'takePhoto', {
  mode: 'album',
  count: 10
}, (result) => {
  const imagePaths = result.images.map(img => img.url);
  compressor.addToQueue(imagePaths);
});
```

## Use Case 3: Adaptive Compression theo Network

```javascript
class AdaptiveImageCompressor {
  async compressForNetwork(imagePath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVNetwork', 'getNetworkType', {}, (network) => {
        let quality, maxWidth;
        
        switch (network.networkType) {
          case 'wifi':
          case '5g':
            quality = 0.9;
            maxWidth = 2048;
            break;
          case '4g':
            quality = 0.7;
            maxWidth = 1920;
            break;
          case '3g':
            quality = 0.5;
            maxWidth = 1280;
            break;
          default:
            quality = 0.3;
            maxWidth = 800;
        }

        window.WindVane.call('WVImage', 'compressImage', {
          src: imagePath,
          quality: quality,
          maxWidth: maxWidth,
          maxHeight: maxWidth * 0.75
        }, (result) => {
          console.log(`Compressed for ${network.networkType}: ${(result.compressedSize / 1024).toFixed(0)}KB`);
          resolve(result);
        }, reject);
      });
    });
  }

  async uploadWithAdaptiveCompression(imagePath) {
    window.WindVane.call('WVUI', 'showLoadingBox', {
      text: 'Đang nén ảnh...'
    });

    try {
      const compressed = await this.compressForNetwork(imagePath);
      
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      window.WindVane.call('WVUI', 'showLoadingBox', {
        text: 'Đang upload...'
      });

      await this.upload(compressed.path);
      
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      window.WindVane.call('WVUIToast', 'toast', {
        message: 'Upload thành công!'
      });
      
    } catch (error) {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      window.WindVane.call('WVUIDialog', 'alert', {
        message: 'Lỗi: ' + error.message
      });
    }
  }

  async upload(imagePath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'uploadFile', {
        url: 'https://api.example.com/upload',
        filePath: imagePath,
        name: 'image'
      }, resolve, reject);
    });
  }
}

// Sử dụng
const adaptiveCompressor = new AdaptiveImageCompressor();

window.WindVane.call('WVCamera', 'takePhoto', {}, (photo) => {
  adaptiveCompressor.uploadWithAdaptiveCompression(photo.url);
});
```

## Best Practices

:::tip Compression Strategy
- **Avatar/Profile**: quality 0.7, 512x512
- **Posts/Feed**: quality 0.8, 1920x1080
- **High-quality**: quality 0.9, 2048x2048
- **Thumbnail**: quality 0.5, 200x200
:::

### 1. Preserve Aspect Ratio

```javascript
function compressPreservingAspect(imagePath, maxDimension = 1920) {
  return new Promise((resolve, reject) => {
    // Get original dimensions first
    window.WindVane.call('WVImage', 'getImageInfo', {
      src: imagePath
    }, (info) => {
      const aspectRatio = info.width / info.height;
      let maxWidth, maxHeight;
      
      if (aspectRatio > 1) {
        // Landscape
        maxWidth = maxDimension;
        maxHeight = maxDimension / aspectRatio;
      } else {
        // Portrait
        maxWidth = maxDimension * aspectRatio;
        maxHeight = maxDimension;
      }

      window.WindVane.call('WVImage', 'compressImage', {
        src: imagePath,
        quality: 0.8,
        maxWidth: Math.round(maxWidth),
        maxHeight: Math.round(maxHeight)
      }, resolve, reject);
    });
  });
}
```

### 2. Quality-based Compression

```javascript
async function compressUntilSize(imagePath, targetSizeKB) {
  let quality = 0.9;
  let result;

  while (quality > 0.3) {
    result = await compress(imagePath, quality);
    
    const sizeKB = result.compressedSize / 1024;
    
    if (sizeKB <= targetSizeKB) {
      console.log(`Achieved ${sizeKB.toFixed(0)}KB at quality ${quality}`);
      return result;
    }
    
    quality -= 0.1;
  }

  console.warn(`Could not reach target size. Final: ${(result.compressedSize / 1024).toFixed(0)}KB`);
  return result;
}

function compress(imagePath, quality) {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVImage', 'compressImage', {
      src: imagePath,
      quality: quality
    }, resolve, reject);
  });
}
```

### 3. Show Compression Stats

```javascript
function showCompressionResult(original, compressed) {
  const savedBytes = original - compressed;
  const savedPercent = (savedBytes / original * 100).toFixed(1);
  
  window.WindVane.call('WVUIToast', 'toast', {
    message: `Giảm ${(savedBytes / 1024).toFixed(0)}KB (${savedPercent}%)`
  });
}
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `INVALID_FORMAT` | Định dạng ảnh không hỗ trợ | Chỉ dùng JPG, PNG |
| `FILE_NOT_FOUND` | Đường dẫn ảnh không tồn tại | Kiểm tra path |
| `OUT_OF_MEMORY` | Ảnh quá lớn | Giảm maxWidth/maxHeight |

## API liên quan

- [takePhoto](./takePhoto) - Chụp ảnh trước khi nén
- [chooseVideo](./chooseVideo) - Video cũng cần nén
- [uploadFile](../D_file_storage/uploadFile) - Upload ảnh đã nén
