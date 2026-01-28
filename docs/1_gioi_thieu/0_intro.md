---
sidebar_position: 0
slug: /
---

# Nền tảng Superapp Tammi

Nền tảng Superapp Tammi được phát triển bởi Viettel Telecom, tập trung vào tính linh hoạt, khả năng mở rộng và trải nghiệm người dùng liền mạch, cho phép tích hợp đa dạng miniapp trong một hệ sinh thái thống nhất.

Một Miniapp Tammi hoạt động như sau, với phần chính của Tammi bao gồm:

- **Môi trường runtime**: Cung cấp không gian thực thi cho các miniapp
- **Bridge API (jsAPI)**: Giao thức giao tiếp giữa miniapp và các tính năng native của thiết bị

<div align="center">

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#EE0033',
    'primaryTextColor': '#FFFFFF',
    'primaryBorderColor': '#EE0033',
    'lineColor': '#44494D',
    'secondaryColor': '#F2F2F2',
    'tertiaryColor': '#B5B4B4',
    'background': '#FFFFFF',
    'mainBkg': '#FFFFFF',
    'secondaryBkg': '#EE0033',
    'tertiaryBkg': '#F2F2F2',
    'activationBorderColor': '#44494D',
    'activationBkgColor': '#F2F2F2',
    'sequenceNumberColor': '#000000',
    'sectionBkgColor': '#F2F2F2',
    'altSectionBkgColor': '#F2F2F2',
    'gridColor': '#B5B4B4',
    'textColor': '#000000',
    'actorTextColor': '#FFFFFF',
    'actorBkg': '#EE0033',
    'actorBorder': '#EE0033',
    'actorLineColor': '#B5B4B4',
    'signalColor': '#44494D',
    'signalTextColor': '#000000',
    'labelBoxBkgColor': '#F2F2F2',
    'labelBoxBorderColor': '#B5B4B4',
    'labelTextColor': '#000000',
    'loopTextColor': '#000000',
    'noteTextColor': '#000000',
    'noteBkgColor': '#F2F2F2',
    'noteBorderColor': '#B5B4B4'
  }
}}%%
sequenceDiagram
    participant MiniApp as 🌐 MiniApp<br/>(WebView)
    participant Bridge as 🔗 JS Bridge
    participant Native as 📱 Native App<br/>(iOS/Android)

    MiniApp->>Bridge: Gọi API (vd: getLocation())
    Bridge->>Native: Chuyển tiếp request
    Native->>Native: Xử lý & truy cập<br/>tài nguyên thiết bị
    Native-->>Bridge: Trả kết quả
    Bridge-->>MiniApp: Callback với data
```

_Hình 1: Luồng giao tiếp giữa MiniApp và Native App thông qua JS Bridge_

</div>

Nền tảng cung cấp nhiều thư viện và API để lựa chọn, tùy thuộc vào nhu cầu của ứng dụng. Bạn có thể xây dựng một miniapp đơn giản chỉ với logic nghiệp vụ cơ bản, hoặc tận dụng Bridge API để truy cập các tính năng native như camera, GPS - mang lại trải nghiệm tương tự native app.

## Bắt đầu phát triển

Sẵn sàng xây dựng miniapp? Chọn phương thức phù hợp với bạn:

| Phương thức                                                  | Mô tả                                        |
| ------------------------------------------------------------ | -------------------------------------------- |
| [**A. Xây mới**](/getting_started/A_xay_moi/index)           | Phát triển miniapp từ đầu - linh hoạt nhất   |
| [**B. Chuyển đổi SPA**](/getting_started/B_chuyen_doi/index) | Chuyển đổi ứng dụng React/Vue/Angular có sẵn |
| [**C. Tích hợp Webview**](/getting_started/C_webview/index)  | Nhúng website có sẵn vào miniapp             |

[Xem chi tiết tại Getting Started](/getting_started/index)
