# bury

埋点

- ajax
- [img](#img)
- [`Navigator.sendBeacon`](#sendbeacon)

## img

img标签携带参数

```js
const img = new Image();
img.src = 'https://localhost/button-01.jpg?x=1&y=2';
```

## sendBeacon

成功率最高，兼容性不好

```js
window.addEventListener('unload', logData, false);

function logData() {
    navigator.sendBeacon("/log", analyticsData);
}
```
