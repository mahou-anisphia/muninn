---
sidebar_position: 2
---

# chooseVideo

Chọn video từ thư viện hoặc quay video mới.

## Cú pháp

```javascript
window.WindVane.call(
  'WVVideo',
  'chooseVideo',
  {
    mode: 'album',         // 'camera', 'album', 'both'
    maxDuration: 60,       // Thời lượng tối đa (giây)
    camera: 'back',        // 'front', 'back'
    compressed: 'true'     // Nén video: 'true' hoặc 'false'
  },
  function(result) {
    console.log('Video URL:', result.tempFilePath);
    console.log('Duration:', result.duration, 'seconds');
  },
  function(error) {
    console.error('Error:', error);
  }
);
```

## Tham số đầu vào

| Tham số | Kiểu | Mặc định | Mô tả |
|---------|------|----------|-------|
| `mode` | `string` | `'both'` | Nguồn video: `camera` (quay mới), `album` (thư viện), `both` |
| `maxDuration` | `number` | `60` | Thời lượng tối đa (giây) |
| `camera` | `string` | `'back'` | Camera: `front` (trước), `back` (sau) |
| `compressed` | `string` | `'true'` | Nén video sau khi quay/chọn |

## Success Callback

```typescript
{
  tempFilePath: string;  // Đường dẫn file local
  duration: number;      // Thời lượng video (giây)
  size: number;          // Kích thước file (bytes)
  width: number;         // Chiều rộng video (px)
  height: number;        // Chiều cao video (px)
}
```

## Fail Callback

| Error Code | Mô tả |
|------------|-------|
| `USER_CANCEL` | User hủy chọn video |
| `NO_PERMISSION` | Không có quyền camera/album |
| `FILE_TOO_LARGE` | File vượt quá giới hạn |
| `DURATION_EXCEEDED` | Video quá dài |

## Ví dụ

### 1. Quay video 30 giây

```javascript
window.WindVane.call('WVVideo', 'chooseVideo', {
  mode: 'camera',
  maxDuration: 30,
  camera: 'back',
  compressed: 'true'
}, (result) => {
  console.log(`Video: ${result.duration}s, ${result.size} bytes`);
  uploadVideo(result.tempFilePath);
}, (error) => {
  if (error.error === 'NO_PERMISSION') {
    alert('Vui lòng cấp quyền Camera trong Settings');
  }
});
```

### 2. Chọn video từ thư viện

```javascript
window.WindVane.call('WVVideo', 'chooseVideo', {
  mode: 'album',
  maxDuration: 120,  // Cho phép video tối đa 2 phút
  compressed: 'true'
}, (result) => {
  // Kiểm tra kích thước
  const sizeMB = result.size / (1024 * 1024);
  if (sizeMB > 50) {
    alert('Video quá lớn. Vui lòng chọn video khác.');
    return;
  }
  
  displayVideoPreview(result.tempFilePath);
}, (error) => {
  console.error('Lỗi chọn video:', error);
});
```

### 3. Upload video với progress

```javascript
async function uploadVideoWithProgress() {
  try {
    // Chọn video
    const video = await new Promise((resolve, reject) => {
      window.WindVane.call('WVVideo', 'chooseVideo', {
        mode: 'album',
        maxDuration: 60,
        compressed: 'true'
      }, resolve, reject);
    });
    
    // Hiển thị thông tin
    const durationMin = Math.floor(video.duration / 60);
    const durationSec = video.duration % 60;
    const sizeMB = (video.size / (1024 * 1024)).toFixed(2);
    
    window.WindVane.call('WVUIToast', 'toast', {
      message: `Video: ${durationMin}:${durationSec}, ${sizeMB}MB`
    });
    
    // Confirm upload
    const confirmed = await new Promise((resolve) => {
      window.WindVane.call('WVUIDialog', 'confirm', {
        title: 'Upload video',
        message: `Upload video ${sizeMB}MB?`,
        okButton: 'Upload',
        cancelButton: 'Hủy'
      }, () => resolve(true), () => resolve(false));
    });
    
    if (!confirmed) return;
    
    // Loading
    window.WindVane.call('WVUI', 'showLoadingBox', {});
    
    // Upload
    const formData = new FormData();
    formData.append('video', video.tempFilePath);
    
    const response = await fetch('/api/upload-video', {
      method: 'POST',
      body: formData
    });
    
    window.WindVane.call('WVUI', 'hideLoadingBox', {});
    
    if (response.ok) {
      window.WindVane.call('WVUIToast', 'toast', {
        message: 'Upload thành công!'
      });
    }
    
  } catch (error) {
    window.WindVane.call('WVUI', 'hideLoadingBox', {});
    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Lỗi upload video'
    });
  }
}
```

### 4. Video review app

```javascript
function recordVideoReview() {
  window.WindVane.call('WVVideo', 'chooseVideo', {
    mode: 'camera',
    maxDuration: 120,  // 2 phút review
    camera: 'front',   // Camera trước (selfie)
    compressed: 'true'
  }, (result) => {
    // Preview trước khi submit
    showVideoPreview(result.tempFilePath, () => {
      // Submit review
      submitReview({
        videoUrl: result.tempFilePath,
        duration: result.duration
      });
    });
  }, (error) => {
    if (error.error !== 'USER_CANCEL') {
      alert('Không thể quay video');
    }
  });
}
```

### 5. React component

```jsx
import { useState } from 'react';

function VideoUploader() {
  const [video, setVideo] = useState(null);
  const [uploading, setUploading] = useState(false);

  const chooseVideo = async () => {
    try {
      const result = await new Promise((resolve, reject) => {
        window.WindVane.call('WVVideo', 'chooseVideo', {
          mode: 'both',
          maxDuration: 60,
          compressed: 'true'
        }, resolve, reject);
      });
      
      setVideo(result);
    } catch (error) {
      if (error.error !== 'USER_CANCEL') {
        alert('Lỗi: ' + error.errorMessage);
      }
    }
  };

  const uploadVideo = async () => {
    if (!video) return;
    
    setUploading(true);
    try {
      const formData = new FormData();
      formData.append('video', video.tempFilePath);
      
      await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      alert('Upload thành công!');
      setVideo(null);
    } catch (error) {
      alert('Lỗi upload');
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <button onClick={chooseVideo}>Chọn Video</button>
      
      {video && (
        <div>
          <p>Thời lượng: {video.duration}s</p>
          <p>Kích thước: {(video.size / (1024 * 1024)).toFixed(2)}MB</p>
          <button onClick={uploadVideo} disabled={uploading}>
            {uploading ? 'Đang upload...' : 'Upload'}
          </button>
        </div>
      )}
    </div>
  );
}
```

## Best Practices

### ✅ Nên làm

- **Nén video**: Luôn set `compressed: 'true'` để giảm kích thước
- **Giới hạn duration**: Set `maxDuration` hợp lý (< 120s)
- **Kiểm tra file size**: Validate trước khi upload
- **Progress indicator**: Hiển thị progress khi upload
- **Preview**: Cho user xem lại video trước khi upload

### ❌ Không nên

- Upload video không nén (rất tốn bandwidth)
- Cho phép video quá dài (> 5 phút)
- Không kiểm tra kích thước file
- Không xử lý lỗi permission

## Use Cases

| Use Case | Cấu hình | Xử lý |
|----------|---------|-------|
| **Video review** | `mode: 'camera', maxDuration: 120, camera: 'front'` | Upload sau preview |
| **Story/Reels** | `mode: 'camera', maxDuration: 30, camera: 'back'` | Auto-upload |
| **Upload từ thư viện** | `mode: 'album', maxDuration: 300` | Validate size trước upload |
| **Video call recording** | `mode: 'camera', camera: 'front'` | Stream hoặc save local |

## Giới hạn

| Giới hạn | Giá trị |
|----------|---------|
| File size tối đa | 100MB (khuyến nghị < 50MB) |
| Thời lượng tối đa | 600s (10 phút) |
| Định dạng hỗ trợ | MP4, MOV, M4V |
| Độ phân giải tối đa | 1920 x 1080 (Full HD) |

## Security & Privacy

:::warning Quyền Camera/Album
API này yêu cầu quyền:
- **Camera**: Để quay video mới
- **Microphone**: Để ghi âm
- **Photo Library**: Để chọn video từ thư viện

User sẽ được hỏi cấp quyền lần đầu tiên sử dụng.
:::

## Performance Tips

- Luôn nén video với `compressed: 'true'`
- Giới hạn `maxDuration` để tránh file quá lớn
- Upload video trong background nếu có thể
- Cache video đã upload để tránh upload lại

## Tương thích

| Nền tảng | Hỗ trợ |
|----------|--------|
| iOS | ✅ Từ iOS 11+ |
| Android | ✅ Từ Android 5.0+ |

## Xem thêm

- [takePhoto](./takePhoto) - Chụp/chọn ảnh
- [uploadFile](../D_file_storage/uploadFile) - Upload file
- [compressVideo](./compressVideo) - Nén video
