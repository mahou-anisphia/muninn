---
sidebar_position: 7
---

# scan (QR/Barcode)

Mở camera để quét mã QR hoặc Barcode.

## Cú pháp

```javascript
window.WindVane.call(
  'WVScan',
  'scan',
  {
    type: 'qr' | 'barcode' | 'all',
    hideAlbum: boolean
  },
  (result) => { /* success */ },
  (error) => { /* fail */ }
);
```

## Tham số đầu vào

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `type` | `string` | Không | Loại mã: `'qr'`, `'barcode'`, `'all'` - Mặc định: `'all'` |
| `hideAlbum` | `boolean` | Không | Ẩn nút chọn ảnh từ album - Mặc định: `false` |

## Kết quả trả về

**Success callback:**

```javascript
{
  content: string,    // Nội dung mã đã quét
  type: string,       // Loại mã: 'QR_CODE', 'CODE_128', 'EAN_13', etc.
  rawData: string     // Dữ liệu thô (hex)
}
```

**Fail callback:**

```javascript
{
  error: string,             // Mã lỗi
  errorMessage: string       // Mô tả lỗi
}
```

## Ví dụ

### 1. Quét QR code cơ bản

```javascript
window.WindVane.call('WVScan', 'scan',
  { type: 'qr' },
  (result) => {
    console.log('QR Content:', result.content);
    console.log('QR Type:', result.type);
    handleQRContent(result.content);
  },
  (error) => {
    console.error('Scan failed:', error);
  }
);
```

### 2. Quét barcode sản phẩm

```javascript
const scanProductBarcode = () => {
  window.WindVane.call('WVScan', 'scan',
    {
      type: 'barcode',
      hideAlbum: true  // Chỉ cho phép quét, không chọn từ album
    },
    (result) => {
      console.log('Barcode:', result.content);
      lookupProduct(result.content);
    },
    (error) => {
      showToast('Không quét được mã sản phẩm');
    }
  );
};

const lookupProduct = async (barcode) => {
  const response = await fetch(`/api/products/${barcode}`);
  const product = await response.json();
  displayProduct(product);
};
```

### 3. Promise wrapper

```javascript
const scan = (type = 'all', hideAlbum = false) => {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVScan', 'scan',
      { type, hideAlbum },
      resolve,
      reject
    );
  });
};

// Sử dụng
async function handleScan() {
  try {
    const result = await scan('qr');
    console.log('Scanned:', result.content);
    
    // Parse QR content
    if (result.content.startsWith('http')) {
      openBrowser(result.content);
    } else if (result.content.startsWith('viettel://')) {
      handleDeeplink(result.content);
    } else {
      showToast('QR Code: ' + result.content);
    }
  } catch (error) {
    console.error('Scan error:', error);
  }
}
```

### 4. React component

```jsx
import { useState } from 'react';

function QRScanner() {
  const [scanning, setScanning] = useState(false);
  const [result, setResult] = useState(null);

  const startScan = async () => {
    setScanning(true);
    try {
      const scanResult = await new Promise((resolve, reject) => {
        window.WindVane.call('WVScan', 'scan',
          { type: 'qr', hideAlbum: false },
          resolve,
          reject
        );
      });
      setResult(scanResult);
      handleQRContent(scanResult.content);
    } catch (error) {
      if (error.error !== 'USER_CANCELLED') {
        alert('Lỗi quét: ' + error.errorMessage);
      }
    } finally {
      setScanning(false);
    }
  };

  const handleQRContent = (content) => {
    // Xử lý theo nội dung QR
    if (content.startsWith('http')) {
      // URL
      window.open(content);
    } else if (content.includes('@')) {
      // Email
      window.location.href = `mailto:${content}`;
    } else {
      // Text thường
      alert('QR Content: ' + content);
    }
  };

  return (
    <div>
      <button onClick={startScan} disabled={scanning}>
        {scanning ? 'Đang quét...' : 'Quét QR Code'}
      </button>
      
      {result && (
        <div>
          <h3>Kết quả:</h3>
          <p>Type: {result.type}</p>
          <p>Content: {result.content}</p>
        </div>
      )}
    </div>
  );
}
```

### 5. Check-in với QR

```javascript
const checkInWithQR = async () => {
  try {
    const result = await scan('qr', true);
    
    // Verify QR format
    if (!result.content.startsWith('CHECKIN:')) {
      throw new Error('QR code không hợp lệ');
    }
    
    const locationId = result.content.replace('CHECKIN:', '');
    
    // Gọi API check-in
    const response = await fetch('/api/checkin', {
      method: 'POST',
      body: JSON.stringify({
        locationId,
        timestamp: Date.now()
      })
    });
    
    if (response.ok) {
      showToast('Check-in thành công!');
    }
  } catch (error) {
    showToast('Lỗi: ' + error.message);
  }
};
```

### 6. Payment QR scanner

```javascript
const scanPaymentQR = async () => {
  try {
    const result = await scan('qr', true);
    
    // Parse payment QR (VietQR format)
    const paymentData = parseVietQR(result.content);
    
    // Confirm before payment
    const confirmed = await showConfirmDialog({
      title: 'Xác nhận thanh toán',
      message: `
        Ngân hàng: ${paymentData.bankName}
        Số tài khoản: ${paymentData.accountNumber}
        Số tiền: ${formatCurrency(paymentData.amount)}
      `
    });
    
    if (confirmed) {
      await processPayment(paymentData);
    }
  } catch (error) {
    showToast('Không thể xử lý QR thanh toán');
  }
};

const parseVietQR = (qrContent) => {
  // Parse VietQR format
  // Format: {bankCode}|{accountNumber}|{amount}|{description}
  const [bankCode, accountNumber, amount, description] = qrContent.split('|');
  
  return {
    bankCode,
    bankName: getBankName(bankCode),
    accountNumber,
    amount: parseFloat(amount),
    description
  };
};
```

## Use Cases

| Use Case | Cấu hình | Xử lý content |
|----------|---------|---------------|
| **Check-in** | `type: 'qr', hideAlbum: true` | Verify prefix, call API |
| **Product lookup** | `type: 'barcode', hideAlbum: true` | Query product DB |
| **URL shortener** | `type: 'qr'` | Open browser if valid URL |
| **Payment** | `type: 'qr', hideAlbum: true` | Parse format, confirm |
| **Contact** | `type: 'qr'` | Parse vCard, add to contacts |

## Best Practices

### 1. Validate QR content

```javascript
const validateQRContent = (content, expectedType) => {
  switch (expectedType) {
    case 'url':
      try {
        new URL(content);
        return true;
      } catch {
        return false;
      }
    
    case 'email':
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(content);
    
    case 'phone':
      return /^[0-9]{10,11}$/.test(content);
    
    case 'checkin':
      return content.startsWith('CHECKIN:');
    
    default:
      return true;
  }
};

// Sử dụng
const result = await scan('qr');
if (!validateQRContent(result.content, 'url')) {
  showToast('QR code không phải là URL hợp lệ');
  return;
}
```

### 2. Error handling

```javascript
const handleScanError = (error) => {
  switch (error.error) {
    case 'PERMISSION_DENIED':
      showDialog({
        title: 'Cần quyền camera',
        message: 'Vui lòng cấp quyền camera trong Settings',
        buttons: [
          { text: 'Đóng' },
          { text: 'Mở Settings', action: () => openSettings('camera') }
        ]
      });
      break;
    
    case 'USER_CANCELLED':
      // Không cần thông báo
      break;
    
    case 'SCAN_FAILED':
      showToast('Không thể quét mã. Vui lòng thử lại.');
      break;
    
    case 'NO_CAMERA':
      showToast('Thiết bị không có camera');
      break;
    
    default:
      showToast('Lỗi: ' + error.errorMessage);
  }
};
```

### 3. Continuous scanning

```javascript
const continuousScan = async (onScan, { maxScans = 10, delay = 1000 }) => {
  let scanCount = 0;
  
  while (scanCount < maxScans) {
    try {
      const result = await scan('qr', true);
      
      // Callback xử lý
      const shouldContinue = await onScan(result);
      
      if (!shouldContinue) {
        break;
      }
      
      scanCount++;
      
      // Delay trước lần quét tiếp
      await new Promise(r => setTimeout(r, delay));
    } catch (error) {
      if (error.error === 'USER_CANCELLED') {
        break;
      }
      console.error('Scan error:', error);
    }
  }
};

// Sử dụng cho inventory scanning
continuousScan(async (result) => {
  await addToInventory(result.content);
  showToast(`Đã thêm: ${result.content}`);
  return true; // Continue scanning
}, { maxScans: 50, delay: 500 });
```

## Lỗi thường gặp

| Mã lỗi | Nguyên nhân | Xử lý |
|--------|-------------|-------|
| `PERMISSION_DENIED` | Chưa cấp quyền camera | Yêu cầu vào Settings |
| `USER_CANCELLED` | User đóng màn hình scan | Không cần xử lý |
| `SCAN_FAILED` | Không nhận diện được mã | Retry hoặc hướng dẫn user |
| `NO_CAMERA` | Thiết bị không có camera | Disable feature |
| `INVALID_QR` | Mã bị hỏng/mờ | Yêu cầu scan lại |

## Barcode types hỗ trợ

| Format | Mô tả | Use case |
|--------|-------|----------|
| **QR_CODE** | QR Code 2D | URL, text, payment |
| **CODE_128** | Barcode 1D | Vận chuyển, kho |
| **CODE_39** | Barcode 1D | Inventory |
| **EAN_13** | Barcode sản phẩm | Retail |
| **EAN_8** | Barcode ngắn | Sản phẩm nhỏ |
| **UPC_A** | Barcode US | US products |

## Quyền yêu cầu

- **Camera**: Cần approve trong Dashboard
- **User consent**: Hiển thị dialog xin quyền lần đầu

:::tip Khuyến nghị
- Luôn validate content sau khi quét
- Cung cấp hướng dẫn rõ ràng cho user (align QR vào khung)
- Xử lý trường hợp QR bị mờ/hỏng
- Cho phép user retry nếu scan thất bại
- Hiển thị preview content trước khi xử lý
:::

## Xem thêm

- [takePhoto](./takePhoto) - Chụp ảnh
- [canIUse](../A_app_base/canIUse) - Kiểm tra quyền camera
