# webpack config

> 项目结构
```
|- dist/ // 打包后的文件存储位置
|--- main.js // 默认打包后的入口文件(webpack4+)
|- src/
|--- index.js // 默认将该文件当作入口文件(webpack4+)
|--- index.html
|- package.json
```

```json
{
  "scripts": {
    "dev": "webpack"
  }
}
```

- src/index.js
```ecmascript 6
console.log(111111);
```

- 执行 **npm run dev** 打包

- src/index.html
```html
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>index</title>
</head>
<body>
  <!-- 引入编译后入口主程序main.js -->
  <script src="../dist/main.js"></script>
</body>
</html>
```

> webpack.config.js
```ecmascript 6
module.exports = {
  mode: 'development' // production
};
```

> webpack-dev-server 实现代码同步更新

安装 webpack-dev-server后修改 **package.json** 的 **scripts**

```json
{
  "scripts": {
    "dev": "webpack-dev-serve"
  }
}
```

再次执行 ``npm run dev`` 打包后，会开启一个服务，打包后不会退出，并刷新代码。


在浏览器里输入``localhost:3000/mai.js``是可以访问到打包后的``dist/main.js``的原因：

虽然在``localhost:3000``里看不到，webpack-dev-server 打包好的 main.js 已经被托管到内存中，可以引用，那么，这样可以减少磁盘打包读写。

那么，要实现代码实时更新，只需要修改 **src/index.html** 里的 **main.js** 引入路径即可
```html
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>index</title>
</head>
<body>
  <!-- 引入编译后入口主程序main.js -->
  <!--<script src="../dist/main.js"></script>-->
  <script src="/main.js"></script>
</body>
</html>
```
