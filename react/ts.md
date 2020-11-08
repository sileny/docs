# react-ts项目

## 创建项目
```
npx create-react-app react-ts --template typescript
```

## 项目规范依赖

引入了 `husky` 对提交的代码进行规范，因此，要先初始化仓库 `git init`

```
yarn add husky lint-staged prettier -D
```

常用的配置如下，通常配置在 `package.json` 里
```
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,json}": [
      "prettier --write",
      "eslint --fix"
    ],
    "src/**/*.{css,scss,less,json,html,md,markdown}": [
      "prettier --write"
    ]
  },
```

## 按需导入

```
yarn add babel-plugin-import -D
```

常用配置如下，在 `package.json` 里

下面以按需导入 `antd` 为例，配置在 `pacakge.json` 里
```
  "babel": {
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd",
          "libraryDirectory": "es",
          "style": true
        }
      ]
    ],
    "presets": [
      "react-app"
    ]
  }
```

## 支持装饰器写法

本项目本身就是 `typescript` 项目，所以，不用配置
```
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


## 开发依赖

样式文件编译
```
yarn add sass -D
yarn add less less-loader -D
```


## 项目文件分析
```
yarn add source-map-explorer -D
```

添加分析 `scripts`
```
"scripts": {
  "analyze": "source-map-explorer build/static/js/main.*"
}
```

## 运行时依赖
```
yarn add react-redux react-router-dom redux
```

`redux` 和 `react-redux` 进行状态管理

`react-router-dom` 路由导航

安装 `types` 文件，支持 `typescript`

```
yarn add @types/react-router-dom @types/react-redux
```


## ui组件

动画效果比较好看
```
yarn add @material-ui/core
yarn add @material-ui/icons
yarn add @material-ui/pickers
yarn add react-swipeable-views @types/react-swipeable-views # 懒加载swipe-views
yarn add react-text-mask react-number-format @types/react-text-mask # 格式化
yarn add react-swipeable-views-utils @types/react-swipeable-views-utils
yarn add react-draggable
```


## 项目文件说明


1、`react-app-env.d.ts` 声明文件

```ts
/// <reference types="react-scripts" />
```

三斜线指令是包含单个XML标签的单行注释。 注释的内容会做为编译器指令使用。

三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在一个语句或声明之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。

```ts
/// <reference path="..." />
/// <reference path="..." />指令是三斜线指令中最常见的一种。 它用于声明文件间的 依赖。
```

三斜线引用告诉编译器在编译过程中要引入的额外的文件。

当使用 `--out` 或 `--outFile` 时，它也可以做为调整输出内容顺序的一种方法。 文件在输出文件内容中的位置与经过预处理后的输入顺序一致。


2、`React.FC`

```
FC = Functional Component
SFC = Stateless Functional Component (已弃用)
```


