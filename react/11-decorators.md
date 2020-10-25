# decorators

支持装饰器模式


## 方式一

```
yarn eject
```
或
```
npm run eject
```

在 `package.json` 文件里添加以下配置
```json
"babel": {
  "plugins": [
    [
      "@babel/plugin-proposal-decorators",
      {
        "legacy": true
      }
    ]
  ],
  "presets": [
    "react-app"
  ]
}
```


## 方式二

1 修改 `package.json`

```json
"scripts": {
  "start": "react-app-rewired start",
  "build": "react-app-rewired build",
  "test": "react-app-rewired test",
  "eject": "react-scripts eject"
}
```

2 安装编译依赖包

在新版中，`@babel/plugin-proposal-decorators` 依赖已经集成，不需要安装，安装 `customize-cra` 和 `react-app-rewired` 即可

```js
yarn add react-app-rewired customize-cra -D
```

在根目录下添加 `config-overrides.js`，与 `package.json` 同级

```js
const { override, addDecoratorsLegacy } = require('customize-cra')
module.exports = override(addDecoratorsLegacy())
```
