---
sidebar_position: 5
---

# isAppsInstalled - Kiểm tra app đã cài

Kiểm tra xem các ứng dụng khác có được cài đặt trên thiết bị hay không.

## API Call

```javascript
const apps = {
  'facebook': { ios: 'fb://', android: 'com.facebook.katana' },
  'zalo': { ios: 'zalo://', android: 'com.zing.zalo' }
};

window.WindVane.call('WVBase', 'isAppsInstalled', apps, (result) => {
  console.log('Facebook installed:', result['facebook']);
  console.log('Zalo installed:', result['zalo']);
});
```

## Tham số

Object với key là tên app, value là:

| Thuộc tính | Kiểu | Bắt buộc | Mô tả |
|-----------|------|----------|-------|
| `ios` | `string` | Có | URL scheme iOS |
| `android` | `string` | Có | Package name Android |

## Success Result

Object với key là tên app, value là `true`/`false`.

## Use Case: Social Share

```javascript
class SocialShareChecker {
  apps = {
    facebook: { ios: 'fb://', android: 'com.facebook.katana' },
    messenger: { ios: 'fb-messenger://', android: 'com.facebook.orca' },
    zalo: { ios: 'zalo://', android: 'com.zing.zalo' }
  };

  async getInstalledApps() {
    return new Promise((resolve) => {
      window.WindVane.call('WVBase', 'isAppsInstalled', this.apps, (result) => {
        const installed = Object.keys(result).filter(app => result[app]);
        resolve(installed);
      });
    });
  }

  async showShareOptions() {
    const installedApps = await this.getInstalledApps();
    
    const buttons = installedApps.map(app => {
      const names = {
        facebook: 'Facebook',
        messenger: 'Messenger',
        zalo: 'Zalo'
      };
      return names[app];
    });

    window.WindVane.call('WVUIActionSheet', 'show', {
      title: 'Chia sẻ qua',
      buttons: buttons
    }, (result) => {
      const selectedApp = installedApps[result.index];
      this.shareViaApp(selectedApp);
    });
  }

  shareViaApp(appName) {
    // Implement share logic
    console.log('Sharing via', appName);
  }
}
```
