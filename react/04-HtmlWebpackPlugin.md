# HtmlWebpackPlugin

```ecmascript 6
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 'src/index.html', // 源文件模板
      filename: 'index.html' // 会被打包到内存中
    })
  ]
};
```

上面的配置会将 ``src/index.html`` 打包到内存中，就像 **config.md** 里说的那样，直接在浏览器里可以直接访问 ``dist/index.html``


浏览器地址栏输入 ``localhost:3000`` 回车即可。


f12即可查看打包到内存中的页面结构，如下，
```html
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>index</title>
</head>
<body>
  <!-- 不需要再次手动引入打包后的文件(因为约定) -->
  <!--<script src="/mai.js"></script>-->
  <script type="text/javascript" src="main.js"></script>
</body>
</html>
```

