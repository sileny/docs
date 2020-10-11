# 使用babel转换在js里写入html标记

```ecmascript 6
// src/index.js

import React from 'react';
import ReactDom from 'react-dom';

const Test = <p>test</p>;

ReactDom.render(<Test/>, document.getElementById('root'));
```

上面的代码并不能被正确地执行,会提示如下错误：
```
Uncaught Error: Module parse failed: Unexpected Token
You may need an appropriate loader to handle this file type 
```

需要按照下面的步骤做：

- 安装包依赖
babel-core babel-loader  babel-plugin-transform-runtime
babel-preset-env babel-preset-stage-0
babel-preset-react

- 创建 .babelrc 文件
```
{
  "presets": ["env", "stage-0", "react"],
  "plugins": ["transform-runtime"]
}
```

- 修改 webpack.config.js
```ecmascript 6
// webpack.config.js

const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  module: { // 处理类型文件
    rules: [
      {test: /\.jsx?$/, use: 'babel-loader', exclude: /node_modules/}
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 'src/index.html', // 源文件模板
      filename: 'index.html' // 会被打包到内存中
    })
  ]
};
```
