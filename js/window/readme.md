# window

## open

`open` 打开一个浏览器窗口

```
const windowObjectReference = window.open(strUrl, strWindowName, [strWindowFeatures]);
```

- `strUrl` 要在新打开的窗口中加载的URL。

- `strWindowName` 新窗口的名称。

- `strWindowFeatures` 一个可选参数，列出新窗口的特征(大小，位置，滚动条等)作为一个 [DOMString](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/open#Position%20and%20size%20features)


```
const windowObjectReference = null; // global variable

function openFFPromotionPopup() {
  if (windowObjectReference == null || windowObjectReference.closed) {
    windowObjectReference = window.open("http://www.spreadfirefox.com/", "PromoteFirefoxWindowName", "resizable,scrollbars,status");
  } else {
    windowObjectReference.focus();
  }
  return false;
}
```

## close

```js
windowObjectReference.close();
```
