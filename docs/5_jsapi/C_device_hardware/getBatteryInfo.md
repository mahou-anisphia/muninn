---
sidebar_position: 8
---

# getBatteryInfo

Lấy thông tin trạng thái pin hiện tại của thiết bị.

## Cú pháp

```javascript
window.WindVane.call(
  'WVBattery',
  'getBatteryInfo',
  {},
  (result) => { /* success */ },
  (error) => { /* fail */ }
);
```

## Kết quả trả về

**Success callback:**

```javascript
{
  level: number,          // Mức pin (0-100)
  isCharging: boolean,    // Đang sạc hay không
  chargingTime: number,   // Thời gian sạc đầy (seconds), Infinity nếu không sạc
  dischargingTime: number // Thời gian hết pin (seconds), Infinity nếu đang sạc
}
```

## Ví dụ

### 1. Lấy thông tin pin cơ bản

```javascript
window.WindVane.call('WVBattery', 'getBatteryInfo', {},
  (result) => {
    console.log(`Pin: ${result.level}%`);
    console.log(`Đang sạc: ${result.isCharging ? 'Có' : 'Không'}`);
  },
  (error) => {
    console.error('Không lấy được thông tin pin:', error);
  }
);
```

### 2. Promise wrapper

```javascript
const getBatteryInfo = () => {
  return new Promise((resolve, reject) => {
    window.WindVane.call('WVBattery', 'getBatteryInfo', {},
      resolve,
      reject
    );
  });
};

// Sử dụng
async function checkBattery() {
  try {
    const battery = await getBatteryInfo();
    console.log('Battery info:', battery);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### 3. Battery indicator component (React)

```jsx
import { useState, useEffect } from 'react';

function BatteryIndicator() {
  const [battery, setBattery] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchBattery = async () => {
      try {
        const result = await new Promise((resolve, reject) => {
          window.WindVane.call('WVBattery', 'getBatteryInfo', {},
            resolve,
            reject
          );
        });
        setBattery(result);
      } catch (error) {
        console.error('Battery error:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchBattery();
    
    // Refresh mỗi 60s
    const interval = setInterval(fetchBattery, 60000);
    return () => clearInterval(interval);
  }, []);

  if (loading) return <div>Loading...</div>;
  if (!battery) return null;

  const getBatteryColor = (level) => {
    if (level > 50) return '#4CAF50';
    if (level > 20) return '#FF9800';
    return '#F44336';
  };

  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <div
        style={{
          width: '40px',
          height: '20px',
          border: '2px solid #333',
          borderRadius: '4px',
          position: 'relative',
          padding: '2px'
        }}
      >
        <div
          style={{
            width: `${battery.level}%`,
            height: '100%',
            backgroundColor: getBatteryColor(battery.level),
            borderRadius: '2px',
            transition: 'width 0.3s'
          }}
        />
        {battery.isCharging && (
          <span style={{
            position: 'absolute',
            top: '50%',
            left: '50%',
            transform: 'translate(-50%, -50%)',
            fontSize: '12px'
          }}>
            ⚡
          </span>
        )}
      </div>
      <span>{battery.level}%</span>
    </div>
  );
}
```

### 4. Low battery warning

```javascript
const checkLowBattery = async () => {
  const battery = await getBatteryInfo();
  
  if (battery.level < 20 && !battery.isCharging) {
    showDialog({
      title: 'Pin yếu',
      message: `Pin còn ${battery.level}%. Vui lòng sạc thiết bị.`,
      type: 'warning'
    });
    return true;
  }
  
  return false;
};

// Sử dụng trước khi thực hiện tác vụ nặng
async function startHeavyTask() {
  const isLowBattery = await checkLowBattery();
  
  if (isLowBattery) {
    const confirmed = await confirm('Pin yếu. Vẫn tiếp tục?');
    if (!confirmed) return;
  }
  
  // Thực hiện tác vụ
  await performHeavyTask();
}
```

### 5. Battery-aware operations

```javascript
class BatteryAwareManager {
  async shouldPerformTask(taskType) {
    const battery = await getBatteryInfo();
    
    // Nếu đang sạc, OK cho mọi tác vụ
    if (battery.isCharging) {
      return true;
    }
    
    // Quyết định dựa trên mức pin
    if (taskType === 'heavy') {
      return battery.level > 30;
    }
    
    if (taskType === 'medium') {
      return battery.level > 15;
    }
    
    return battery.level > 5;
  }

  async adjustQuality() {
    const battery = await getBatteryInfo();
    
    if (battery.level < 20) {
      return 'low'; // Giảm chất lượng để tiết kiệm pin
    }
    
    if (battery.isCharging || battery.level > 50) {
      return 'high';
    }
    
    return 'medium';
  }
}

// Sử dụng
const batteryManager = new BatteryAwareManager();

async function uploadPhotos(photos) {
  const canUpload = await batteryManager.shouldPerformTask('heavy');
  
  if (!canUpload) {
    showToast('Pin yếu. Upload sẽ tiếp tục khi sạc thiết bị.');
    // Queue để upload sau
    queueForLater(photos);
    return;
  }
  
  const quality = await batteryManager.adjustQuality();
  await uploadWithQuality(photos, quality);
}
```

### 6. Battery monitoring hook

```jsx
function useBatteryMonitor(threshold = 20) {
  const [battery, setBattery] = useState(null);
  const [isLow, setIsLow] = useState(false);

  useEffect(() => {
    const checkBattery = async () => {
      try {
        const result = await getBatteryInfo();
        setBattery(result);
        setIsLow(result.level < threshold && !result.isCharging);
      } catch (error) {
        console.error('Battery monitor error:', error);
      }
    };

    checkBattery();
    const interval = setInterval(checkBattery, 30000); // Check mỗi 30s
    
    return () => clearInterval(interval);
  }, [threshold]);

  return { battery, isLow };
}

// Component sử dụng
function AppWithBatteryWarning() {
  const { battery, isLow } = useBatteryMonitor(15);

  return (
    <div>
      {isLow && (
        <div style={{
          background: '#FF9800',
          padding: '8px',
          textAlign: 'center'
        }}>
          ⚠️ Pin yếu ({battery?.level}%). Vui lòng sạc thiết bị.
        </div>
      )}
      
      {/* App content */}
    </div>
  );
}
```

## Use Cases

| Use Case | Logic | Hành động |
|----------|-------|-----------|
| **Video upload** | `level < 30 && !isCharging` | Defer upload |
| **Camera quality** | `level < 20` | Reduce quality |
| **Background sync** | `level < 15 && !isCharging` | Pause sync |
| **Gaming** | `level < 10` | Show warning |
| **Heavy computation** | `level < 25 && !isCharging` | Ask confirmation |

## Best Practices

### 1. Polling interval

```javascript
// ❌ Không nên poll quá thường xuyên
const interval = setInterval(checkBattery, 1000); // Quá nhanh

// ✅ Nên poll hợp lý
const interval = setInterval(checkBattery, 30000); // 30s là đủ
```

### 2. Battery-aware caching

```javascript
const getCacheStrategy = async () => {
  const battery = await getBatteryInfo();
  
  if (battery.level < 20 && !battery.isCharging) {
    return {
      ttl: 3600,        // Cache lâu hơn
      prefetch: false,  // Không prefetch
      quality: 'low'    // Chất lượng thấp
    };
  }
  
  return {
    ttl: 300,
    prefetch: true,
    quality: 'high'
  };
};
```

### 3. Graceful degradation

```javascript
const performTaskWithBatteryCheck = async (task) => {
  try {
    const battery = await getBatteryInfo();
    
    if (battery.level < 10) {
      throw new Error('Pin quá yếu');
    }
    
    return await task();
  } catch (error) {
    if (error.message === 'Pin quá yếu') {
      showToast('Pin yếu. Tác vụ bị hủy.');
      return null;
    }
    throw error;
  }
};
```

## Lưu ý

:::warning Giới hạn
- API có thể không hoạt động trên emulator (trả về giá trị mặc định)
- Giá trị `chargingTime` và `dischargingTime` không chính xác 100%
- Chỉ nên dùng để tham khảo, không dùng cho logic quan trọng
:::

:::tip Khuyến nghị
- Poll interval: 30-60 giây
- Hiển thị warning khi pin < 20%
- Giảm chất lượng khi pin < 15%
- Defer heavy tasks khi pin < 30% và không sạc
- Không block UX vì pin yếu, chỉ cảnh báo
:::

## Quyền yêu cầu

- **Battery**: Không cần approve, là Ordinary Permission

## Xem thêm

- [getSystemInfo](./getSystemInfo) - Thông tin hệ thống
- [getNetworkType](./getNetworkType) - Loại mạng
