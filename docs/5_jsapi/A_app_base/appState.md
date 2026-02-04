---
sidebar_position: 1
---

# appState - Trạng thái miniapp

Trả về trạng thái hiện tại của miniapp (active/background/inactive).

## API Call

```javascript
window.WindVane.call('WVApplication', 'appState', {}, (result) => {
  console.log('App state:', result.state);
});
```

## Success Result

| Thuộc tính | Kiểu | Mô tả |
|-----------|------|-------|
| `state` | `string` | Trạng thái: `active`, `background`, `inactive` |

## Use Case: Pause Video on Background

```javascript
class VideoLifecycle {
  constructor(videoElement) {
    this.video = videoElement;
    this.monitorState();
  }

  monitorState() {
    window.WindVane.call('WVApplication', 'onAppStateChange', {}, (result) => {
      if (result.state === 'background') {
        this.video.pause();
      } else if (result.state === 'active') {
        // Optional: resume video
        // this.video.play();
      }
    });
  }
}
```
