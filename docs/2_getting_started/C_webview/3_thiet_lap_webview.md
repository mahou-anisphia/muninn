---
sidebar_position: 3
---

# Thiết lập Webview

Sau khi miniapp đã chạy thành công, bước tiếp theo là thay thế nội dung mặc định bằng webview để load website của bạn.

## Bước 1: Chỉnh sửa App.tsx

Mở file `src/App.tsx` và thay thế toàn bộ nội dung bằng code sau:

```tsx title="src/App.tsx"
import { useEffect, useState } from "react";

function App() {
  const [webviewUrl, setWebviewUrl] = useState<string>("");

  useEffect(() => {
    // Thay đổi URL này thành website của bạn
    setWebviewUrl("https://viettel.com.vn/vi/");
  }, []);

  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      {webviewUrl && (
        <iframe
          src={webviewUrl}
          style={{
            width: "100%",
            height: "100%",
            border: "none",
          }}
          title="Webview"
        />
      )}
    </div>
  );
}

export default App;
```

:::tip Thay đổi URL
Thay `https://viettel.com.vn/vi/` bằng URL website thực tế của bạn.
:::

## Bước 2: Preview lại trên Android

Sau khi lưu file, quay lại **Miniapp Extension** và chọn **Preview on Android Device/Emulator** để xem kết quả.

Website của bạn sẽ được hiển thị trong miniapp.

---

## Nâng cao: Tích hợp SSO với Tammi

Trong một số trường hợp, bạn muốn:

- Đồng bộ tài khoản người dùng với Tammi Account
- Sử dụng SSO của Tammi để xác thực

<details>
<summary><strong>Hướng dẫn tích hợp SSO</strong></summary>

### Bước 1: Tạo hàm getAuthCode

Thêm file `src/utils/auth.ts`:

```typescript title="src/utils/auth.ts"
export const getAuthCode = (
  appId: string,
  scopes: string[] = ["USER_NAME", "USER_PHONE_NUMBER"],
): Promise<string> => {
  return new Promise((resolve, reject) => {
    if (!window.WindVane) {
      reject(
        new Error(
          "WindVane is not available. Please run in Mini App environment.",
        ),
      );
      return;
    }

    const params = {
      appId,
      scopes,
    };

    window.WindVane.call(
      "wv",
      "getAuthCode",
      params,
      (result: { authCode: string }) => {
        if (result?.authCode) {
          resolve(result.authCode);
        } else {
          reject(new Error("No auth code returned"));
        }
      },
      (error: any) => {
        reject(new Error(JSON.stringify(error) || "Failed to get auth code"));
      },
    );
  });
};
```

### Bước 2: Cập nhật App.tsx

```tsx title="src/App.tsx"
import { useEffect, useState } from "react";
import { getAuthCode } from "./utils/auth";

function App() {
  const [webviewUrl, setWebviewUrl] = useState<string>("");
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string>("");

  useEffect(() => {
    const initializeApp = async () => {
      try {
        // Lấy authCode từ Tammi SSO
        const authCode = await getAuthCode("YOUR_APP_ID");

        // Truyền authCode vào webview URL
        const baseUrl = "https://your-website.com";
        setWebviewUrl(`${baseUrl}?authCode=${authCode}`);
      } catch (err) {
        setError(err instanceof Error ? err.message : "Unknown error");
      } finally {
        setLoading(false);
      }
    };

    initializeApp();
  }, []);

  if (loading) {
    return <div>Đang xác thực...</div>;
  }

  if (error) {
    return <div>Lỗi: {error}</div>;
  }

  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      {webviewUrl && (
        <iframe
          src={webviewUrl}
          style={{
            width: "100%",
            height: "100%",
            border: "none",
          }}
          title="Webview"
        />
      )}
    </div>
  );
}

export default App;
```

### Bước 3: Xử lý authCode phía backend

Backend của bạn cần gọi API Tammi để exchange `authCode` lấy thông tin user:

```
POST https://api.tammi.vn/api/v1/auth/oauth/get-access-token
Auth: Basic Auth (sử dụng CLIENT_ID và CLIENT_SECRET)
{
	"authCode": "YOUR_AUTH_CODE"
}
```

:::warning Lưu ý quan trọng

- `getAuthCode` **chỉ hoạt động** khi miniapp đã deploy lên Tammi, không hoạt động ở chế độ local preview
- `CLIENT_ID` và `CLIENT_SECRET` được cấp sau khi đăng kí miniapp trên Tammi và khai báo nhu cầu sử dụng SSO với Viettel.
  :::

</details>

## Tiếp theo

Webview đã được thiết lập. Xem [Bước tiếp theo](./buoc_tiep_theo) để tìm hiểu về quy trình deploy và các tài liệu tham khảo.
