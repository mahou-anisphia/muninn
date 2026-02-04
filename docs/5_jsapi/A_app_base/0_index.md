---
sidebar_position: 0
id: index
---

# App & Base APIs

Các API cơ bản để quản lý trạng thái ứng dụng, kiểm tra tính năng, và thao tác hệ thống cơ bản.

## Danh sách APIs

| API | Mô tả | Quyền yêu cầu |
|-----|-------|---------------|
| [appState](./appState) | Lấy trạng thái hiện tại của miniapp | Không |
| [canIUse](./canIUse) | Kiểm tra tính khả dụng của API | Không |
| [closeMiniApp](./closeMiniApp) | Đóng miniapp và quay về Superapp | Không |
| [copyToClipboard](./copyToClipboard) | Sao chép văn bản vào clipboard | Không |
| [getNotificationSettings](./getNotificationSettings) | Kiểm tra trạng thái quyền thông báo | Có |
| [isAppsInstalled](./isAppsInstalled) | Kiểm tra app khác có cài đặt không | Có |
| [openBrowser](./openBrowser) | Mở URL bằng trình duyệt mặc định | Không |
| [openSettings](./openSettings) | Mở trang cài đặt hệ thống | Không |
| [setBackgroundColor](./setBackgroundColor) | Đặt màu nền container | Không |

## Use Cases phổ biến

### 1. Kiểm tra API trước khi sử dụng

```javascript
// Kiểm tra xem có thể dùng camera không
window.WindVane.call('WVBase', 'canIUse', { api: 'takePhoto' }, (result) => {
  if (result.result) {
    // Có thể sử dụng - tiếp tục gọi camera API
  } else {
    // Không khả dụng - hiển thị thông báo hoặc fallback
  }
});
```

### 2. Xử lý lifecycle của app

```javascript
// Lắng nghe khi app chuyển trạng thái
window.WindVane.call('WVApplication', 'appState', {}, (result) => {
  if (result.state === 'background') {
    // App vào background - tạm dừng timer, video...
  } else if (result.state === 'active') {
    // App active trở lại - tiếp tục
  }
});
```

### 3. Copy & Share

```javascript
const shareReferralCode = (code) => {
  window.WindVane.call('WVBase', 'copyToClipboard', 
    { text: code }, 
    () => {
      alert('Đã copy mã giới thiệu!');
    }
  );
};
```

## Lưu ý quan trọng

:::warning Không lạm dụng openBrowser
`openBrowser` sẽ đưa user rời khỏi miniapp. Chỉ dùng khi thực sự cần thiết (ví dụ: link điều khoản, chính sách).
:::

:::tip Sử dụng canIUse
Luôn dùng `canIUse` để kiểm tra quyền trước khi gọi các API yêu cầu phê duyệt.
:::
