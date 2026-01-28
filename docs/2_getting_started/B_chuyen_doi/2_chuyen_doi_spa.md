---
sidebar_position: 2
---

# Chuyển đổi SPA

Sau khi thiết lập môi trường, bạn cần thực hiện một số thay đổi trong project SPA để tương thích với nền tảng miniapp.

## Bước 1: Kiểm tra package.json

Mở file `package.json` và đảm bảo có các trường sau:

### 1.1. Trường `version`

```json title="package.json"
{
  "name": "your-app-name",
  "version": "1.0.0",  // Bắt buộc phải có
  ...
}
```

:::warning Lưu ý
Nếu `package.json` không có trường `version`, hãy thêm vào. Miniapp platform yêu cầu trường này để quản lý phiên bản.
:::

### 1.2. Scripts `start` và `build`

Đảm bảo có 2 scripts sau:

```json title="package.json"
{
  "scripts": {
    "start": "...", // Chạy development server
    "build": "..." // Build production bundle
  }
}
```

| Script  | Mục đích                        | Ví dụ                                      |
| ------- | ------------------------------- | ------------------------------------------ |
| `start` | Chạy app ở localhost để preview | `react-scripts start`, `vite`, `vue serve` |
| `build` | Build static files để deploy    | `react-scripts build`, `vite build`        |

## Bước 2: Thêm WindVane SDK

WindVane là thư viện Bridge API (jsAPI) cho phép miniapp giao tiếp với native layer của Tammi Superapp.

Thêm script sau vào file `index.html` (trong thẻ `<head>`):

```html title="public/index.html hoặc index.html"
<head>
  <!-- Các thẻ khác -->
  <script src="https://g.alicdn.com/mtb/lib-windvane/3.0.0/windvane.js"></script>
</head>
```

:::info WindVane là gì?
WindVane cung cấp object `window.WindVane` để gọi các Bridge API như:

- Lấy thông tin user (SSO)
- Truy cập camera, GPS
- Hiển thị native UI (toast, dialog)
- Và nhiều tính năng khác

Xem [Tài liệu Bridge API](../jsapi/) để biết chi tiết.
:::

## Bước 3: Chuyển sang Hash Routing

Miniapp yêu cầu sử dụng **hash-based routing** (`/#/path`) thay vì history-based routing (`/path`).

### React Router

```jsx title="src/App.jsx hoặc src/index.jsx"
import { HashRouter } from "react-router-dom";

// Thay BrowserRouter bằng HashRouter
function App() {
  return <HashRouter>{/* Routes của bạn */}</HashRouter>;
}
```

### Vue Router

```js title="src/router/index.js"
import { createRouter, createWebHashHistory } from 'vue-router';

const router = createRouter({
  // Thay createWebHistory bằng createWebHashHistory
  history: createWebHashHistory(),
  routes: [...]
});
```

### React Router v5 (legacy)

```jsx
import { HashRouter } from "react-router-dom";

ReactDOM.render(
  <HashRouter>
    <App />
  </HashRouter>,
  document.getElementById("root"),
);
```

## Bước 4: Kiểm tra cấu hình build

Đảm bảo output build là **relative path** để miniapp có thể load đúng assets.

### Create React App

Thêm trường `homepage` vào `package.json`:

```json title="package.json"
{
  "homepage": "./"
}
```

### Vite

```js title="vite.config.js"
export default defineConfig({
  base: "./",
  // ...
});
```

### Vue CLI

```js title="vue.config.js"
module.exports = {
  publicPath: "./",
};
```

## Checklist trước khi tiếp tục

Trước khi chạy miniapp, hãy đảm bảo:

- [ ] `package.json` có trường `version`
- [ ] Có scripts `npm run start` và `npm run build`
- [ ] Đã thêm WindVane SDK vào `index.html`
- [ ] Đã chuyển sang hash routing (nếu có routing)
- [ ] Đã cấu hình relative path cho build output

## Tiếp theo

Project đã sẵn sàng. Tiếp tục với [Khởi chạy Miniapp](./khoi_chay_miniapp) để preview trên thiết bị.
