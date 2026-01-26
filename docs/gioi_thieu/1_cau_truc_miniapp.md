---
sidebar_position: 1
---

# Cấu trúc Miniapp

## Miniapp là gì?

Miniapp là một **Single Page Application (SPA)** chạy dưới dạng static webpage trong sandbox của SuperApp Tammi. Các công nghệ web tiêu chuẩn như routing, redirect và browser APIs đều hoạt động bình thường. Ngoài ra, miniapp có thể gọi **Bridge API (jsAPI)** để truy cập các tính năng native của thiết bị — điều mà web app thông thường không làm được.

Có thể hình dung miniapp như một nền tảng nằm giữa web thuần túy và React Native: mạnh mẽ hơn web app nhờ truy cập được tính năng native, nhưng vẫn chưa hoàn toàn linh hoạt như native app do mọi tương tác đều phải thông qua lớp superapp.

:::tip
Vì Bridge API (jsAPI) là tùy chọn, một SPA thuần có thể chạy trực tiếp trên nền tảng miniapp mà không cần chỉnh sửa nhiều.
:::
