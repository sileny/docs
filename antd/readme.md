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

本项目分让 `antd` 主题配置可以参考[官方地址](https://ant.design/docs/react/customize-theme-cn)

**注意：**
- `less-loader` 的处理范围不要过滤掉 `node_modules` 下的 `antd` 包。
- `lessOptions` 的配置写法在 `less-loader@6.0.0` 里支持。

完整的 `webpack.config.js` 参考[这个项目](https://github.com/sileny/react-app-hoc/blob/main/config/webpack.config.js)


## env

多次使用到 `'#1DA57A'`，可以将该变量放在 `.env` 文件里，具体的配置如下，
```
REACT_APP_primaryColor=#f00
```
然后，在 `webpack.config.js` 里声明一个变量 `const theme = process.env.REACT_APP_primaryColor;`，将 `'#1DA57A'` 替换为 `theme`，然后，重新编译即可

>环境变量命名规则：`REACT_APP_`为前缀

- `.env`: 所有环境生效
- `.env.local`: 重写本地变量。除了 `test` 环境都生效
- `.env.development`, `.env.test`, `.env.production`: 指定环境生效
- `.env.development.local`, `.env.test.local`, `.env.production.local`: 重写本地变量.

优先级如下
* `npm start`: `.env.development.local` > `.env.development` > `.env.local` > `.env`
* `npm run build`: `.env.production.local` > `.env.production` > `.env.local` > `.env` 
* `npm test`: `.env.test.local` > `.env.test` > `.env`


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

> 不需要导入 `antd` 的任何样式，`babel-plugin-import` 会按需提取组件依赖
