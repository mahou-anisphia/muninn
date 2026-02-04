---
sidebar_position: 6
---

# previewImage - Xem ảnh toàn màn hình

Hiển thị ảnh trong viewer toàn màn hình với zoom và swipe.

## API Call

```javascript
window.WindVane.call('WVImage', 'previewImage', {
  current: 0,
  urls: [
    'https://example.com/image1.jpg',
    'https://example.com/image2.jpg',
    '/storage/emulated/0/Download/photo.jpg'
  ]
}, () => {
  console.log('Preview opened');
});
```

## Tham số

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `current` | `number` | Không | Index ảnh hiển thị đầu tiên, mặc định 0 |
| `urls` | `string[]` | Có | Danh sách URL hoặc local path |

## Use Case 1: Photo Gallery

```javascript
class PhotoGallery {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.photos = [];
  }

  setPhotos(photoUrls) {
    this.photos = photoUrls;
    this.render();
  }

  render() {
    this.container.innerHTML = this.photos.map((url, index) => `
      <div class="gallery-item" onclick="gallery.preview(${index})">
        <img src="${url}" alt="Photo ${index + 1}">
      </div>
    `).join('');
  }

  preview(startIndex) {
    window.WindVane.call('WVImage', 'previewImage', {
      current: startIndex,
      urls: this.photos
    });
  }
}

// Sử dụng
const gallery = new PhotoGallery('#photo-gallery');
gallery.setPhotos([
  'https://picsum.photos/800/600?random=1',
  'https://picsum.photos/800/600?random=2',
  'https://picsum.photos/800/600?random=3'
]);
```

## Use Case 2: Product Images Viewer

```javascript
class ProductImageViewer {
  constructor(productId) {
    this.productId = productId;
    this.images = [];
  }

  async loadProductImages() {
    const response = await fetch(`/api/products/${this.productId}`);
    const product = await response.json();
    this.images = product.images;
    
    this.renderThumbnails();
  }

  renderThumbnails() {
    const container = document.getElementById('product-images');
    container.innerHTML = `
      <div class="main-image" onclick="productViewer.preview(0)">
        <img src="${this.images[0]}" alt="Main product image">
        <div class="zoom-hint">Tap để phóng to</div>
      </div>
      <div class="thumbnails">
        ${this.images.map((url, index) => `
          <img src="${url}" onclick="productViewer.preview(${index})">
        `).join('')}
      </div>
    `;
  }

  preview(startIndex) {
    window.WindVane.call('WVImage', 'previewImage', {
      current: startIndex,
      urls: this.images
    });
  }

  addImage(imageUrl) {
    this.images.push(imageUrl);
    this.renderThumbnails();
  }
}

// Sử dụng
const productViewer = new ProductImageViewer('product-123');
await productViewer.loadProductImages();
```

## Use Case 3: Chat Image Preview

```javascript
class ChatImagePreview {
  constructor() {
    this.messageImages = new Map(); // messageId -> imageUrls
  }

  addMessage(messageId, imageUrls) {
    this.messageImages.set(messageId, imageUrls);
  }

  previewMessageImages(messageId, startIndex = 0) {
    const urls = this.messageImages.get(messageId);
    
    if (!urls || urls.length === 0) {
      console.warn('No images in this message');
      return;
    }

    window.WindVane.call('WVImage', 'previewImage', {
      current: startIndex,
      urls: urls
    });
  }

  renderMessage(messageId, imageUrls) {
    this.addMessage(messageId, imageUrls);
    
    const html = imageUrls.map((url, index) => `
      <div class="chat-image" onclick="chatPreview.previewMessageImages('${messageId}', ${index})">
        <img src="${url}" loading="lazy">
        ${imageUrls.length > 1 ? `<span class="image-count">${index + 1}/${imageUrls.length}</span>` : ''}
      </div>
    `).join('');
    
    return html;
  }
}

// Sử dụng
const chatPreview = new ChatImagePreview();

// Khi nhận message mới
const message = {
  id: 'msg-123',
  images: [
    'https://example.com/chat1.jpg',
    'https://example.com/chat2.jpg'
  ]
};

document.getElementById('messages').innerHTML += `
  <div class="message">
    <div class="message-images">
      ${chatPreview.renderMessage(message.id, message.images)}
    </div>
  </div>
`;
```

## Use Case 4: Before/After Comparison

```javascript
class BeforeAfterViewer {
  constructor(beforeUrl, afterUrl) {
    this.beforeUrl = beforeUrl;
    this.afterUrl = afterUrl;
  }

  render(containerId) {
    const container = document.getElementById(containerId);
    container.innerHTML = `
      <div class="before-after">
        <div class="image-container">
          <img src="${this.beforeUrl}" onclick="beforeAfter.preview(0)">
          <span class="label">Trước</span>
        </div>
        <div class="image-container">
          <img src="${this.afterUrl}" onclick="beforeAfter.preview(1)">
          <span class="label">Sau</span>
        </div>
      </div>
      <button onclick="beforeAfter.preview(0)" class="btn-compare">
        So sánh toàn màn hình
      </button>
    `;
  }

  preview(startIndex = 0) {
    window.WindVane.call('WVImage', 'previewImage', {
      current: startIndex,
      urls: [this.beforeUrl, this.afterUrl]
    });
  }
}

// Sử dụng
const beforeAfter = new BeforeAfterViewer(
  'https://example.com/before.jpg',
  'https://example.com/after.jpg'
);
beforeAfter.render('comparison-container');
```

## Best Practices

:::tip Image Loading
Sử dụng `loading="lazy"` cho thumbnails để cải thiện performance.
:::

### 1. Preload Images

```javascript
function preloadImages(urls) {
  urls.forEach(url => {
    const img = new Image();
    img.src = url;
  });
}

// Preload trước khi preview
preloadImages(photoUrls);
window.WindVane.call('WVImage', 'previewImage', {
  urls: photoUrls
});
```

### 2. Mix Local và Remote URLs

```javascript
const mixedUrls = [
  'https://example.com/remote.jpg',           // Remote
  '/storage/emulated/0/Download/local.jpg',   // Local
  'https://cdn.example.com/image.jpg'         // CDN
];

window.WindVane.call('WVImage', 'previewImage', {
  current: 0,
  urls: mixedUrls
});
```

### 3. Error Handling

```javascript
function safePreview(urls, startIndex = 0) {
  if (!urls || urls.length === 0) {
    window.WindVane.call('WVUIToast', 'toast', {
      message: 'Không có ảnh để hiển thị'
    });
    return;
  }

  window.WindVane.call('WVImage', 'previewImage', {
    current: Math.max(0, Math.min(startIndex, urls.length - 1)),
    urls: urls
  });
}
```

## Tính năng viewer

Viewer mặc định hỗ trợ:

| Tính năng | Mô tả |
|----------|-------|
| **Pinch to zoom** | Phóng to/thu nhỏ bằng 2 ngón |
| **Swipe** | Vuốt sang trái/phải để xem ảnh khác |
| **Double tap zoom** | Chạm 2 lần để phóng to nhanh |
| **Image counter** | Hiển thị số thứ tự ảnh (vd: 2/5) |
| **Close button** | Nút X để đóng viewer |

## API liên quan

- [takePhoto](./takePhoto) - Chụp ảnh để preview
- [compressImage](./compressImage) - Nén ảnh trước khi lưu
