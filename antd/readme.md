# antd

- [1.eject和按需加载](#babel-import)
- [2.定制主题](#theme)

## babel-import

由于项目已经使用 `yarn eject` 暴露了配置，只需要做以下配置即可

安装依赖
```
yarn add babel-plugin-import -D
```
```
npm i babel-plugin-import -D
```

在 `package.json` 添加如下配置
```json
"babel": {
  "plugins": [
    [
      "import",
      {
        "libraryName": "antd",
        "libraryDirectory": "lib",
        "style": true
      }
    ]
  ],
  "presets": [
    "react-app"
  ]
}
```

>`"style": true`。可以大大减少构建后的包的尺寸，具体能减少多少，取决于实际的包依赖。更多配置参考[这里](https://github.com/ant-design/babel-plugin-import)

## theme


安装依赖
```
yarn add babel-plugin-import less-loader less -D
```
```
npm i babel-plugin-import less-loader less -D
```

## webpack.config

匹配 `less` 文件正则

```js
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;
```

添加 `loader` 配置

```js
{
  test: lessRegex,
  exclude: lessModuleRegex,
  include: /node_modules/,
  use: [{
    loader: 'style-loader'
  }, {
    loader: 'css-loader' // translates CSS into CommonJS
  }, {
    loader: 'less-loader',
    options: {
      lessOptions: { // 如果使用less-loader@5，请移除 lessOptions 这一级直接配置选项。
        modifyVars: {
          'primary-color': '#1DA57A',
          'link-color': '#1DA57A',
          'border-radius-base': '2px'
        },
        javascriptEnabled: true
      }
    }
  }]
},
{
  test: lessModuleRegex,
  use: [{
    loader: 'style-loader'
  }, {
    loader: 'css-loader' // translates CSS into CommonJS
  }, {
    loader: 'less-loader',
    options: {
      lessOptions: { // 如果使用less-loader@5，请移除 lessOptions 这一级直接配置选项。
        modifyVars: {
          'primary-color': '#1DA57A',
          'link-color': '#1DA57A',
          'border-radius-base': '2px'
        },
        javascriptEnabled: true
      }
    }
  }]
},
```

>已经将 `primary-color` 和 `link-color` 的颜色修改为 `#1DA57A`

**注意**
可以将主题色放在环境变量文件里

完整的 `webpack.config.js` 参考[这个项目](https://github.com/sileny/react-app-hoc/blob/main/config/webpack.config.js)

## 按需引入

```js
import React from 'react';
import './App.scss';
import { Button } from 'antd';
import withCopyright from './decorators/withCopyright';

function App() {
  return <div className="App">App<Button type="primary">Button</Button></div>;
}

export default withCopyright(App);
```
