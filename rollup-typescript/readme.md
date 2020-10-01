# rollup-typescript

- [项目结构](#project)
- [tsconfig](#tsconfig)
- [rollup](#rollup)
- [package.json](#package.json)

## project

项目结构：
```
|dist/                      -- 打包文件所在文件夹
|src/                       -- 源码
|.gitattributes
|.gitignore                 -- 忽略文件或文件夹
|.prettierrc
|LICENSE
|babel.config.js            -- ts/es6转换为浏览器运行的版本
|package.json               -- 包依赖
|rollup.config.js           -- roll配置
|tsconfig.json              -- typescript编译配置
```

## tsconfig

`tsconfig.json`
```tsconfig.json
{
  "compilerOptions": {
    "allowUnreachableCode": false, // 不报告执行不到的代码错误。
    "allowUnusedLabels": false, // 不报告未使用的标签错误
    "alwaysStrict": true, // 以严格模式解析并为每个源文件生成 "use strict"语句
    "experimentalDecorators": true, // 启用实验性的ES装饰器
    "sourceMap": false, // 是否生成map文件
    "module": "ES2015", // 指定生成哪个模块系统代码
    "target": "ESNext", // 编译的目标是什么版本的,
    "noImplicitAny": false, // 是否默认禁用 any
    "removeComments": true, // 是否移除注释
    "rootDir": "src",
    "outDir": "dist", // 输出目录
    "declaration": true, // 是否自动创建类型声明文件
    "declarationDir": "dist", // 类型声明文件的输出目录
    "allowJs": false, // 允许编译javascript文件。
    "lib": [ // 编译过程中需要引入的库文件的列表
      "dom",
      "es5",
      "es6",
      "es2015",
      "es2016",
      "es2017",
      "es2018"
    ],
    "types": ["node"],
    "moduleResolution": "node",

    "noUnusedLocals": true,
    "strictNullChecks": true,
    "noImplicitThis": true,
    "noImplicitReturns": true,
    "strict": true,
    "isolatedModules": false,

    "resolveJsonModule": true,
    "esModuleInterop": true
  },
  // 指定一个匹配列表（属于自动指定该路径下的所有ts相关文件）
  "include": [
    "src/**/*"
  ],
  // 指定一个排除列表（include的反向操作）
  "exclude": [
    "node_modules"
//    "dist"
  ]
}
```

## rollup

```js
import path from 'path'
import replace from '@rollup/plugin-replace';
import babel from '@rollup/plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import json from '@rollup/plugin-json';
import { terser } from 'rollup-plugin-terser';
import typescript from 'rollup-plugin-typescript2';

const pkg = require('./package.json');
const { author, email } = pkg;

const banner = `/*!
  * ${ pkg.name } v${ pkg.version }
  * (c) ${ new Date().getFullYear() }
  * author: ${ author }
  * email: ${ email }
  * @license MIT
  */`;

const outputConfigs = {
  // each file name has the format: `dist/${name}.${format}.js`
  // Global and Browser ESM builds inlines everything so that they can be used alone.
  cjs: {
    file: pkg.main,
    format: `cjs`,
  },
  global: {
    file: pkg.unpkg,
    format: `iife`,
  },
  esm: {
    file: pkg.module,
    format: `esm`,
  },
  umd: {
    file: pkg.browser,
    format: 'umd',
    name: 'index'
  }
};

export default Object.keys(outputConfigs).map(format =>
  createConfig(format, outputConfigs[format])
)

function createConfig(format, output) {
  if (!output) {
    console.log(require('chalk').yellow(`invalid format: "${ format }"`));
    process.exit(1)
  }

  output.banner = banner;
  // http://rollupjs.org/guide/en/#outputexternallivebindings
  output.externalLiveBindings = false;

  // 告诉 Rollup jquery 模块的id等同于 $ 变量
  output.globals = { jquery: '$' };

  const isGlobalBuild = format === 'global';
  const isRawESMBuild = format === 'esm';
  const isNodeBuild = format === 'cjs';

  if (isGlobalBuild) output.name = 'TypeHelper';

  const tsPlugin = typescript({
    check: true,
    tsconfig: path.resolve(__dirname, 'tsconfig.json'),
    tsconfigOverride: {},
  });

  const mode = process.env.NODE_ENV;
  const isProd = mode === 'production';

  return {
    input: `src/index.ts`,
    // 参考https://www.rollupjs.com/guide/big-list-of-options/#%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BDcore-functionality
    external: [],
    plugins: [
      tsPlugin,
      createReplacePlugin(
        isGlobalBuild || isRawESMBuild,
        isGlobalBuild,
        isNodeBuild
      ),
      isProd && terser(),
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**',
        extensions: ['.ts']
      })
    ],
    output
  }
}

function createReplacePlugin(isBrowserBuild, isGlobalBuild, isNodeBuild) {
  const replacements = {
    __COMMIT__: `"${ process.env.COMMIT }"`,
    __VERSION__: `"${ pkg.version }"`,
    __BROWSER__: isBrowserBuild,
    __GLOBAL__: isGlobalBuild,
    // is targeting Node (SSR)?
    __NODE_JS__: isNodeBuild,
  };
  // allow inline overrides like
  //__RUNTIME_COMPILE__=true yarn build
  Object.keys(replacements).forEach(key => {
    if (key in process.env) {
      replacements[key] = process.env[key]
    }
  });
  return replace(replacements)
}

```

## package.json

```json
{
  "name": "package-name",
  "version": "1.0.0",
  "description": "",
  "main": "dist/index.cjs.js",
  "unpkg": "dist/index.global.js",
  "module": "dist/index.esm.js",
  "browser": "dist/index.js",
  "scripts": {
    "dev": "rimraf dist && rollup -c --environment NODE_ENV:development",
    "build": "rimraf dist && rollup -c --environment NODE_ENV:production",
    "lint": "prettier --write --parser typescript src/**/*.ts"
  },
  "engines": {
    "node": ">=10.0.0"
  },
  "devDependencies": {
    "@babel/core": "^7.11.6",
    "@babel/preset-env": "^7.11.5",
    "@babel/preset-typescript": "^7.10.4",
    "@rollup/plugin-babel": "^5.2.1",
    "@rollup/plugin-commonjs": "^15.0.0",
    "@rollup/plugin-json": "^4.1.0",
    "@rollup/plugin-node-resolve": "^9.0.0",
    "@rollup/plugin-replace": "^2.3.3",
    "lint-staged": "^10.4.0",
    "prettier": "^2.1.2",
    "rimraf": "^3.0.2",
    "rollup": "^2.27.1",
    "rollup-plugin-terser": "^7.0.2",
    "rollup-plugin-typescript2": "^0.27.3",
    "typescript": "^4.0.3"
  },
  "author": "sunsilent",
  "email": "sunsilently@outlook.com",
  "files": [
    "dist"
  ],
  "license": "MIT",
  "dependencies": {},
  "typings": "dist/index.d.ts",
  "gitHooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "node scripts/verifyCommit.js"
  },
  "lint-staged": {
    "src/**/*.js": [
      "prettier --write",
      "eslint --fix"
    ],
    "src/**/*.ts?(x)": [
      "prettier --parser=typescript --write"
    ],
    "src/**/*.{jsx,json,css,scss,vue}": [
      "prettier --write"
    ]
  }
}
```
