---
sidebar_position: 8
---

# recordVideo - Quay video

Mở camera để quay video.

## API Call

```javascript
window.WindVane.call('WVVideo', 'recordVideo', {
  maxDuration: 60,
  camera: 'back'
}, (result) => {
  console.log('Video path:', result.tempFilePath);
  console.log('Duration:', result.duration);
  console.log('Size:', result.size);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `maxDuration` | `number` | Không | Thời lượng tối đa (giây), mặc định 60 |
| `camera` | `string` | Không | `'front'` hoặc `'back'`, mặc định `'back'` |

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `tempFilePath` | `string` | Đường dẫn video đã quay |
| `duration` | `number` | Thời lượng video (ms) |
| `size` | `number` | Kích thước file (bytes) |
| `width` | `number` | Chiều rộng video (px) |
| `height` | `number` | Chiều cao video (px) |

## Use Case 1: Video Review Recorder

```javascript
class VideoReviewRecorder {
  constructor(maxDuration = 30) {
    this.maxDuration = maxDuration;
  }

  async recordReview() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVUIDialog', 'alert', {
        title: 'Quay video đánh giá',
        message: `Hãy chia sẻ trải nghiệm của bạn (tối đa ${this.maxDuration}s)`
      }, () => {
        window.WindVane.call('WVVideo', 'recordVideo', {
          maxDuration: this.maxDuration,
          camera: 'front'
        }, async (result) => {
          const videoData = await this.processVideo(result);
          resolve(videoData);
        }, reject);
      });
    });
  }

  async processVideo(result) {
    const sizeKB = result.size / 1024;
    const durationSec = result.duration / 1000;

    console.log(`Recorded ${durationSec}s video (${sizeKB.toFixed(0)}KB)`);

    // Check file size
    if (result.size > 10 * 1024 * 1024) { // 10MB
      window.WindVane.call('WVUIToast', 'toast', {
        message: 'Video quá lớn, đang nén...'
      });
      
      // Compress if needed
      return await this.compressVideo(result.tempFilePath);
    }

    return result;
  }

  async compressVideo(videoPath) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVVideo', 'compressVideo', {
        src: videoPath,
        quality: 'medium'
      }, resolve, reject);
    });
  }

  async uploadReview(videoPath, productId) {
    window.WindVane.call('WVUI', 'showLoadingBox', {
      text: 'Đang upload...'
    });

    try {
      const response = await fetch('/api/reviews/video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          productId: productId,
          videoPath: videoPath
        })
      });

      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      
      if (response.ok) {
        window.WindVane.call('WVUIToast', 'toast', {
          message: 'Đã gửi đánh giá thành công!'
        });
      }
    } catch (error) {
      window.WindVane.call('WVUI', 'hideLoadingBox', {});
      window.WindVane.call('WVUIDialog', 'alert', {
        message: 'Lỗi upload: ' + error.message
      });
    }
  }
}

// Sử dụng
const recorder = new VideoReviewRecorder(30);
const video = await recorder.recordReview();
await recorder.uploadReview(video.tempFilePath, 'product-123');
```

## Use Case 2: Video Message Sender

```javascript
class VideoMessageSender {
  constructor(chatId) {
    this.chatId = chatId;
  }

  async recordAndSend() {
    try {
      const video = await this.recordVideo();
      
      // Show preview
      const shouldSend = await this.previewVideo(video);
      
      if (shouldSend) {
        await this.sendVideo(video);
      }
    } catch (error) {
      console.error('Error:', error);
    }
  }

  async recordVideo() {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVVideo', 'recordVideo', {
        maxDuration: 15,
        camera: 'front'
      }, resolve, reject);
    });
  }

  async previewVideo(video) {
    return new Promise((resolve) => {
      // Show video preview
      const preview = document.createElement('video');
      preview.src = video.tempFilePath;
      preview.controls = true;
      
      window.WindVane.call('WVUIDialog', 'confirm', {
        title: 'Gửi video này?',
        message: `Thời lượng: ${(video.duration / 1000).toFixed(0)}s`,
        okbutton: 'Gửi',
        cancelbutton: 'Quay lại'
      }, () => resolve(true), () => resolve(false));
    });
  }

  async sendVideo(video) {
    window.WindVane.call('WVUI', 'showLoadingBox', {
      text: 'Đang gửi...'
    });

    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'uploadFile', {
        url: `/api/chats/${this.chatId}/video`,
        filePath: video.tempFilePath,
        name: 'video'
      }, (result) => {
        window.WindVane.call('WVUI', 'hideLoadingBox', {});
        window.WindVane.call('WVUIToast', 'toast', {
          message: 'Đã gửi video'
        });
        resolve(result);
      }, (error) => {
        window.WindVane.call('WVUI', 'hideLoadingBox', {});
        reject(error);
      });
    });
  }
}

// Sử dụng
const messageSender = new VideoMessageSender('chat-456');
await messageSender.recordAndSend();
```

## Use Case 3: Tutorial Video Creator

```javascript
class TutorialVideoCreator {
  constructor() {
    this.clips = [];
  }

  async recordClip(stepNumber, maxDuration = 20) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVUIDialog', 'alert', {
        title: `Bước ${stepNumber}`,
        message: `Quay video hướng dẫn bước ${stepNumber}`
      }, () => {
        window.WindVane.call('WVVideo', 'recordVideo', {
          maxDuration: maxDuration,
          camera: 'back'
        }, (result) => {
          this.clips.push({
            step: stepNumber,
            path: result.tempFilePath,
            duration: result.duration
          });
          
          window.WindVane.call('WVUIToast', 'toast', {
            message: `Đã quay bước ${stepNumber}`
          });
          
          resolve(result);
        }, reject);
      });
    });
  }

  async createTutorial(steps) {
    for (let i = 0; i < steps.length; i++) {
      await this.recordClip(i + 1, steps[i].maxDuration);
      
      // Ask if want to continue
      if (i < steps.length - 1) {
        const shouldContinue = await this.askContinue(i + 2);
        if (!shouldContinue) break;
      }
    }

    return this.clips;
  }

  async askContinue(nextStep) {
    return new Promise((resolve) => {
      window.WindVane.call('WVUIDialog', 'confirm', {
        message: `Tiếp tục quay bước ${nextStep}?`,
        okbutton: 'Tiếp tục',
        cancelbutton: 'Hoàn thành'
      }, () => resolve(true), () => resolve(false));
    });
  }

  getTotalDuration() {
    return this.clips.reduce((total, clip) => total + clip.duration, 0);
  }

  async uploadTutorial(tutorialName) {
    const totalDuration = this.getTotalDuration();
    
    console.log(`Uploading ${this.clips.length} clips, total: ${(totalDuration / 1000).toFixed(0)}s`);

    // Upload all clips
    for (const clip of this.clips) {
      await this.uploadClip(clip, tutorialName);
    }

    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Đã upload tutorial hoàn chỉnh'
    });
  }

  async uploadClip(clip, tutorialName) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'uploadFile', {
        url: '/api/tutorials/upload',
        filePath: clip.path,
        name: 'clip',
        formData: {
          tutorialName: tutorialName,
          step: clip.step.toString()
        }
      }, resolve, reject);
    });
  }
}

// Sử dụng
const creator = new TutorialVideoCreator();

const steps = [
  { maxDuration: 20 }, // Bước 1
  { maxDuration: 30 }, // Bước 2
  { maxDuration: 15 }  // Bước 3
];

const clips = await creator.createTutorial(steps);
await creator.uploadTutorial('How to Cook Pho');
```

## Best Practices

:::warning File Size
Video có thể rất lớn. Luôn kiểm tra kích thước và nén nếu cần trước khi upload.
:::

### 1. Duration Validation

```javascript
function validateVideoDuration(result, minDuration = 3000) {
  if (result.duration < minDuration) {
    window.WindVane.call('WVUIDialog', 'alert', {
      message: `Video quá ngắn. Tối thiểu ${minDuration / 1000}s`
    });
    return false;
  }
  return true;
}

window.WindVane.call('WVVideo', 'recordVideo', {}, (result) => {
  if (validateVideoDuration(result, 5000)) {
    // Process video
  }
});
```

### 2. Size Warning

```javascript
function checkVideoSize(result) {
  const sizeMB = result.size / 1024 / 1024;
  
  if (sizeMB > 50) {
    window.WindVane.call('WVUIDialog', 'alert', {
      message: `Video ${sizeMB.toFixed(1)}MB quá lớn. Vui lòng quay video ngắn hơn.`
    });
    return false;
  }
  
  if (sizeMB > 10) {
    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Video lớn, có thể upload lâu'
    });
  }
  
  return true;
}
```

### 3. Network-aware Upload

```javascript
async function uploadVideoSmart(videoPath) {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVNetwork', 'getNetworkType', {}, (network) => {
      if (network.networkType === 'none') {
        window.WindVane.call('WVUIDialog', 'alert', {
          message: 'Không có kết nối mạng'
        });
        reject(new Error('No network'));
        return;
      }

      if (network.networkType === '2g' || network.networkType === '3g') {
        window.WindVane.call('WVUIDialog', 'confirm', {
          message: 'Bạn đang dùng mạng di động. Tiếp tục upload?',
          okbutton: 'Upload',
          cancelbutton: 'Hủy'
        }, () => {
          uploadFile(videoPath, resolve, reject);
        }, () => reject(new Error('User cancelled')));
      } else {
        uploadFile(videoPath, resolve, reject);
      }
    });
  });
}

function uploadFile(path, resolve, reject) {
  window.WindVane.call('WVFile', 'uploadFile', {
    url: '/api/upload',
    filePath: path
  }, resolve, reject);
}
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| `PERMISSION_DENIED` | Không có quyền camera | Xin quyền camera/microphone |
| `USER_CANCELLED` | User hủy quay video | Xử lý gracefully |
| `INSUFFICIENT_STORAGE` | Không đủ bộ nhớ | Thông báo dọn dẹp bộ nhớ |

## API liên quan

- [chooseVideo](./chooseVideo) - Chọn video từ thư viện
- [uploadFile](../D_file_storage/uploadFile) - Upload video
