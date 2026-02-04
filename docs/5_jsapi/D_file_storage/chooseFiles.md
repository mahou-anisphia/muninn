---
sidebar_position: 1
---

# chooseFiles - Chọn file từ thiết bị

Mở trình chọn file hệ thống để user chọn file.

## API Call

```javascript
window.WindVane.call('WVFile', 'chooseFiles', {
  count: 3,
  type: 'all' // 'all', 'image', 'video', 'file'
}, (result) => {
  console.log('Selected files:', result.files);
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `count` | `number` | Không | Số file tối đa, mặc định 1 |
| `type` | `string` | Không | Loại file: `all`, `image`, `video`, `file` |

## Success Result

```javascript
{
  files: [
    {
      path: '/storage/emulated/0/Download/document.pdf',
      size: 1024000,
      name: 'document.pdf',
      type: 'application/pdf'
    }
  ]
}
```

## Use Case: Document Upload

```javascript
class DocumentUploader {
  async selectAndUpload() {
    window.WindVane.call('WVFile', 'chooseFiles', {
      count: 5,
      type: 'file'
    }, async (result) => {
      for (const file of result.files) {
        if (file.size > 10 * 1024 * 1024) {
          alert(`File ${file.name} quá lớn (> 10MB)`);
          continue;
        }
        
        await this.uploadFile(file);
      }
    });
  }

  async uploadFile(file) {
    return new Promise((resolve, reject) => {
      window.WindVane.call('WVFile', 'uploadFile', {
        url: 'https://api.example.com/upload',
        filePath: file.path,
        name: 'file'
      }, (result) => {
        window.WindVane.call('WVUIToast', 'toast', {
          message: `Đã upload ${file.name}`
        });
        resolve(result);
      }, reject);
    });
  }
}

// Sử dụng
const uploader = new DocumentUploader();
uploader.selectAndUpload();
```

## API liên quan

- [uploadFile](./uploadFile) - Upload file đã chọn
- [readFile](./readFile) - Đọc nội dung file
