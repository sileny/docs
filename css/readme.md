# css

- [turn角度](#turn)
- [svg](#svg)
- [形状](shape)
- [修正chrome浏览器记录密码样式](#autofill)
- [vmin字体尺寸](#vmin)

## turn

`0.25turn` 为 `90deg`

## autofill

缓解 `webkit` 内核浏览器的表单记录填充样式
```css
input:-webkit-autofill,
textarea:-webkit-autofill,
select:-webkit-autofill,
:-webkit-autofill,
:-webkit-autofill:hover,
:-webkit-autofill:focus,
:-webkit-autofill:active {
  background-color: transparent !important;
  background-image: none !important;
  -webkit-box-shadow: 0 0 3px 100px transparent inset !important;
  -webkit-text-fill-color: $--font-color-light !important;
  transition: background-color 5000s ease-in-out 0s;
}
```

## vmin

字体尺寸

相对于视口的宽度或高度中较小的那个。其中最小的那个被均分为 `100` 单位的 `vmin`

`vh` 和 `vm` 总是与视口的高度和宽度有关，与之不同的，`vmin` 和 `vmax` 是与这次宽度和高度的最大值或最小值有关，取决于哪个更大和更小。例如，如果浏览器设置为 `1100px` 宽、`700px` 高，`1vmin` 会是 `7px`,`1vmax` 为 `11px`。然而，如果宽度设置为 `800px`，高度设置为 `1080px`，`1vmin` 将会等于 `8px` 而 `1vmax` 将会是 `10.8px`

```
h1 {
	font-size: 8vm;
	font-size: 8vmin;
}
```

如果视口的宽度是 `300mm`，高度是 `200mm`，那么上述代码中 `h1` 元素的字号将为 `16mm`，即 `(8x200)/100`，因为高度比宽度要小，所以计算的时候相对于高度

兼容性
```
font-size: 12px;	/*IE6-8*/
font-size: 5vm;		/*IE9*/
font-size: 5vmin;	/*其他浏览器*/
```

