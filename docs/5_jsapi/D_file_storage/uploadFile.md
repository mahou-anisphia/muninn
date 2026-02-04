---
sidebar_position: 3
---

# uploadFile - Upload file lên server

Upload file từ local storage lên server.

## API Call

```javascript
window.WindVane.call('WVFile', 'uploadFile', {
  url: 'https://api.example.com/upload',
  filePath: '/storage/emulated/0/Download/photo.jpg',
  name: 'file',
  formData: {
    userId: '123',
    category: 'avatar'
  }
}, (result) => {
  console.log('Upload success:', result.data);
}, (error) => {
  console.error('Upload failed:', error);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `url` | `string` | Có | URL endpoint |
| `filePath` | `string` | Có | Đường dẫn file local |
| `name` | `string` | Có | Tên field trong form |
| `formData` | `object` | Không | Dữ liệu form thêm vào |
| `header` | `object` | Không | HTTP headers |

## Use Case: Progress Tracking

```javascript
class UploadWithProgress {
  uploadFile(filePath) {
    return new Promise((resolve, reject) => {
      let uploadTask = window.WindVane.call('WVFile', 'uploadFile', {
        url: 'https://api.example.com/upload',
        filePath: filePath,
        name: 'file'
      }, resolve, reject);

      // Track progress
      uploadTask.onProgressUpdate((progress) => {
        const percent = Math.round(progress.totalBytesSent / progress.totalBytesExpectedToSend * 100);
        document.getElementById('progress').style.width = `${percent}%`;
        document.getElementById('percent').textContent = `${percent}%`;
      });

      // Allow cancel
      document.getElementById('cancel-btn').onclick = () => {
        uploadTask.abort();
        reject(new Error('User cancelled'));
      };
    });
  }
}
