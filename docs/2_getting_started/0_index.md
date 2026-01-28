---
sidebar_position: 0
id: index
---

# Getting Started

Chào mừng bạn đến với hướng dẫn phát triển Miniapp Tammi. Tùy thuộc vào tình trạng hiện tại của ứng dụng, hãy chọn một trong ba phương thức tích hợp sau:

## Chọn phương thức phù hợp

| Phương thức             | Mô tả                            | Khi nào sử dụng                                   |
| ----------------------- | -------------------------------- | ------------------------------------------------- |
| **A. Xây mới**          | Phát triển MiniApp từ đầu        | Bắt đầu dự án mới, muốn tận dụng đầy đủ tính năng |
| **B. Chuyển đổi SPA**   | Chuyển đổi ứng dụng SPA có sẵn   | Đã có React/Vue app, muốn chuyển thành Miniapp    |
| **C. Tích hợp Webview** | Nhúng website có sẵn vào Miniapp | Có website hoàn chỉnh, muốn tích hợp nhanh        |

---

## A. Xây mới (Quick Start)

Đây là phương thức **linh hoạt nhất**, cho phép tận dụng toàn bộ tính năng của nền tảng MiniApp.

**Ưu điểm:**

- Truy cập đầy đủ Bridge API
- Tối ưu hiệu năng và trải nghiệm người dùng
- Hỗ trợ kỹ thuật đầy đủ từ đội ngũ phát triển

[Bắt đầu xây mới](./A_xay_moi/index)

---

## B. Chuyển đổi SPA

Phù hợp khi bạn đã có sẵn ứng dụng Single Page Application (SPA) và muốn chuyển đổi thành MiniApp.

**Ưu điểm:**

- Truy cập đầy đủ Bridge API
- Tái sử dụng code có sẵn

**Yêu cầu:**

- Sử dụng **hash-based routing** (`/#/path`)
- Framework hỗ trợ: **React**, **Vue**

[Bắt đầu chuyển đổi SPA](./B_chuyen_doi/index)

---

## C. Tích hợp Webview

Phù hợp khi bạn có website/webapp hoàn chỉnh và muốn tích hợp nhanh với ít thay đổi nhất.

**Ưu điểm:**

- Không cần thay đổi code của ứng dụng gốc
- Hỗ trợ mọi công nghệ web

**Hạn chế:**

- Bridge API chỉ có thể gọi **tại thời điểm khởi tạo** MiniApp
- Không thể gọi Bridge API trong runtime của webview
- Dữ liệu chỉ có thể truyền vào webview một lần (ví dụ: `authCode` qua SSO)

[Bắt đầu tích hợp Webview](./C_webview/index)

---

## Lưu ý quan trọng

:::warning Giới hạn về Routing
MiniApp chỉ hỗ trợ **hash-based routing**. Nếu ứng dụng SPA của bạn sử dụng history-based routing, bạn cần chuyển sang hash-based routing hoặc sử dụng phương thức **Tích hợp Webview**.
:::

:::info Khuyến nghị
Nếu ứng dụng của bạn cần sử dụng **nhiều Bridge API**, chúng tôi khuyến nghị phương thức **Xây mới** để đảm bảo trải nghiệm tốt nhất.
:::
