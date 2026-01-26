---
sidebar_position: 4
---

# Lựa chọn phương thức tích hợp

Tùy thuộc vào tình trạng hiện tại của ứng dụng, bạn có thể chọn một trong ba phương thức tích hợp sau:

| Phương thức        | Mô tả                                              | Mức độ tích hợp Bridge API |
| ------------------ | -------------------------------------------------- | -------------------------- |
| **Xây mới**        | Phát triển MiniApp từ đầu sử dụng SDK              | Đầy đủ                     |
| **Chuyển đổi SPA** | Chuyển đổi ứng dụng SPA có sẵn thành MiniApp       | Đầy đủ                     |
| **Nhúng Webview**  | Nhúng website có sẵn vào MiniApp thông qua webview | Hạn chế                    |

## So sánh chi tiết

### 1. Xây mới

Đây là phương thức **linh hoạt nhất**, cho phép tận dụng toàn bộ tính năng của nền tảng MiniApp.

- ✅ Truy cập đầy đủ Bridge API
- ✅ Tối ưu hiệu năng và trải nghiệm người dùng
- ✅ Hỗ trợ kỹ thuật đầy đủ từ đội ngũ phát triển

### 2. Chuyển đổi SPA

Phù hợp khi bạn đã có sẵn ứng dụng Single Page Application (SPA) và muốn chuyển đổi thành MiniApp.

- ✅ Truy cập đầy đủ Bridge API
- ✅ Tái sử dụng code có sẵn
- ⚠️ Bắt buộc sử dụng **hash-based routing** (`/#/path`)
- ⚠️ Chỉ hỗ trợ chính thức: **Vue**, **React**, **Angular 15+**

### 3. Nhúng Webview

Phù hợp khi bạn có website/webapp hoàn chỉnh và muốn tích hợp nhanh với ít thay đổi nhất.

- ✅ Không cần thay đổi code của ứng dụng gốc
- ✅ Hỗ trợ mọi công nghệ web
- ⚠️ Bridge API chỉ có thể gọi **tại thời điểm khởi tạo** MiniApp
- ⚠️ Không thể gọi Bridge API trong runtime của webview
- ⚠️ Dữ liệu chỉ có thể truyền vào webview một lần (ví dụ: `authCode` qua SSO)

## Lưu ý quan trọng

:::warning Giới hạn về Routing
MiniApp chỉ hỗ trợ **hash-based routing**. Nếu ứng dụng SPA của bạn sử dụng history-based routing, bạn cần chuyển sang hash-based routing hoặc sử dụng phương thức nhúng Webview.
:::

:::info Khuyến nghị
Nếu ứng dụng của bạn cần sử dụng **nhiều Bridge API**, chúng tôi khuyến nghị phương thức **Xây mới** thay vì chuyển đổi SPA hoặc nhúng Webview để đảm bảo trải nghiệm tốt nhất.
:::

## Bắt đầu từ đâu?

Chọn hướng dẫn phù hợp với phương thức tích hợp của bạn:

- 🚀 **Xây mới** → [Hướng dẫn Quick Start](/quick_start/index)
- 🔄 **Chuyển đổi SPA** → [Hướng dẫn chuyển đổi SPA](/chuyen_doi_spa/index)
- 🌐 **Nhúng Webview** → [Hướng dẫn tích hợp Webview](/tich_hop_webview/index)
