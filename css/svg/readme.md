# 通过img引入的svg如何改颜色?

- [修改svg标签](#svg)
- [img](#img)

## svg

```scss
svg {
  fill: currentColor;  //currentColor为css变量，自动读取当前元素颜色
}
```

## img

此时css对svg文件无法生效，此时要用到 `CSS3` 滤镜 `filter` 中的 `drop-shadow`，通过生产一个可指定颜色的阴影放置于svg的位置，并将原始svg移出视线

```html
<img src="logo.svg">
```

```scss
img {
  position: relative;
  left: -143px;
  filter: drop-shadow($--logo-color 143px 0); // 143px 是图片的宽度，也是需要偏移的位置
}
```
