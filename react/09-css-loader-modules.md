# css enable modules

### basic

> webpack.config.js
```ecmascript 6
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  module: { // 处理类型文件
    rules: [
      {test: /\.jsx?$/, use: 'babel-loader', exclude: /node_modules/},
      {
        test: /\.s?css?$/,
        // use: ['style-loader', 'sass-loader', 'css-loader?modules'/* 表示启用模块化 */],
        use: [
          'style-loader',
          'sass-loader',
          'css-loader?modules&localIdentName=[path][name]-[local]-[hash:8]'
        ],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 'src/index.html', // 源文件模板
      filename: 'index.html' // 会被打包到内存中(访问 localhost:3000/index.html)
    })
  ],
  resolve: {
    extensions: ['.js', '.jsx', '.json'], // 使其支持jsx文件查找(导入时可以省略`.jsx`)
    alias: {
      '@': path.resolve(__dirname, './src') // @指向的是`react-app/src`
    }
  }
};
```

> How to use

- Test.jsx
```ecmascript 6
import React from 'react';

import styles from './test.css';

export default class Test extends React.Component {
  render(){
    return (
      <div className={styles.test}>test</div>
    )
  }
};
```

- test.css
```css
.test {
  color: #f00;
}
```

### 进阶

#### 全局和局部样式
```
(:global)(.test) {
  color: blue;
}

// 或者

(:global) {
  .test1 { color: #0f0; }
  .test2 { color: #00f; }
}

// 全局样式写法和之前一样
import React, { Component } from 'react';
import styles from "./test.css";
export default class Test extends Component {
  render() {
    return (<div className="test1">测试</div>)
  }
}
```

