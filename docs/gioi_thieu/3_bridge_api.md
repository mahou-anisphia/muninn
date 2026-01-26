---
sidebar_position: 3
---

# Bridge API (jsAPI) và Sample Code

## Bridge API cho phép bạn làm gì?

Bridge API (jsAPI) là cầu nối giữa miniapp và các tính năng native của thiết bị. Nhờ đó, miniapp có thể:

| Khả năng                | Ví dụ use case                                   |
| ----------------------- | ------------------------------------------------ |
| **Xác thực người dùng** | Đăng nhập tự động qua tài khoản Tammi (SSO)      |
| **Truy cập phần cứng**  | Camera (scan QR), GPS (bản đồ), Bluetooth (IoT)  |
| **Tùy chỉnh giao diện** | Thay đổi navigation bar, hiển thị loading, toast |
| **Điều hướng**          | Mở Mini App khác trong hệ sinh thái Tammi        |
| **Lưu trữ**             | Lưu dữ liệu local trên thiết bị                  |

:::tip Web app thuần có thể chạy mà không cần Bridge API
Bridge API là **tùy chọn**. Nếu miniapp chỉ cần logic nghiệp vụ cơ bản và không cần truy cập phần cứng, bạn hoàn toàn có thể bỏ qua phần này.
:::

## Kiến trúc Bridge API

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#EE0033',
    'primaryTextColor': '#FFFFFF',
    'primaryBorderColor': '#EE0033',
    'lineColor': '#44494D',
    'secondaryColor': '#F2F2F2',
    'tertiaryColor': '#B5B4B4'
  }
}}%%
flowchart TD
    A["Mini App<br/>(JavaScript)"] -->|"WindVane.call()"| B["Bridge API Layer"]
    B -->|"Chuyển tiếp"| C["Native SDK<br/>(iOS/Android)"]
    C -->|"Truy cập"| D["Tính năng thiết bị<br/>(Camera, GPS, Storage...)"]

    style A fill:#EE0033,color:#FFFFFF
    style B fill:#F2F2F2,color:#000000
    style C fill:#FFFFFF,stroke:#EE0033,color:#000000
    style D fill:#E8F5E9,stroke:#44494D,color:#000000
```

**Luồng xử lý:**

1. Miniapp gọi `WindVane.call(module, method, params)`
2. Bridge chuyển request sang native layer
3. Native thực thi (yêu cầu quyền nếu cần)
4. Kết quả trả về qua callback `success` hoặc `fail`

## Trường hợp đặc biệt: SDK Integration

:::caution Yêu cầu tư vấn từ Viettel
Trong một số trường hợp **hiếm gặp**, miniapp cần tích hợp SDK của bên thứ ba (ví dụ: SDK thanh toán đặc thù, analytics platform) **trực tiếp vào native layer** của Tammi Superapp.

**Luồng:** Miniapp → Bridge API → **SDK đã tích hợp sẵn trong Superapp** → Thiết bị

**Điều kiện:**

- SDK phải được Viettel xét duyệt về bảo mật và hiệu năng
- Chỉ áp dụng khi **không thể** xử lý qua backend API thông thường
- Thời gian tích hợp phụ thuộc chu kỳ release của Superapp (thường 2-4 tuần)

**Khi nào cần:** Khi SDK yêu cầu quyền truy cập native (ví dụ: SDK xác thực sinh trắc học, SDK VoIP) mà Bridge API hiện tại không hỗ trợ.

Nếu miniapp của bạn thuộc trường hợp này, liên hệ Viettel để được tư vấn và đánh giá khả thi.
:::

## Sample Code đầy đủ

Repository mẫu demo **toàn bộ Bridge API** có sẵn:

👉 **https://github.com/mahou-anisphia/miniapp-sample-code**

Repo này chứa ví dụ thực tế cho từng API: xác thực, camera, GPS, navigation, storage, UI components, có thể khởi chạy như sau:

1. Clone repo về máy:

```bash title="Terminal"
git clone https://github.com/mahou-anisphia/miniapp-sample-code.git
```

2. Di chuyển vào thư mục project:

```bash title="Terminal"
cd miniapp-sample-code
```

3. Cài đặt dependencies:

```bash title="Terminal"
npm install
```

4. Chạy sample code:

```bash title="Terminal"
npm start
```

:::warning API đặc biệt
`getAuthCode` (xác thực SSO) là API riêng của Tammi, **không hoạt động** ở chế độ local preview. API này chỉ hoạt động sau khi deploy lên nền tảng.
:::

## Tiếp theo

- **Chi tiết từng API**: Xem [Tài liệu jsAPI](../jsapi/) để tìm hiểu tham số, quyền yêu cầu, và use case
- **Bắt đầu phát triển**: Xem [Quick Start](../quick_start/index) để chạy miniapp đầu tiên
- **Tích hợp SSO**: Xem [Cơ chế SSO](../sso/) để hiểu luồng xác thực người dùng

## Hỗ trợ

| Mảng         | Đầu mối      | Email                 |
| ------------ | ------------ | --------------------- |
| Backend, SSO | Hà Anh Vũ    | vuha13@viettel.com.vn |
| Android      | Kiều Văn Bảo | baokv2@viettel.com.vn |
