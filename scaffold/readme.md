# 脚手架基本配置

- [基础配置](#基础配置)
- [项目规范配置](#项目规范配置)
- [项目配置](#项目配置)


## 基础配置

- [polyfill](#polyfill)
- [babel](#babel)
- [打包优化](#打包优化)
- [样式配置](#样式配置)
- [文件清理](#文件清理)
- [cdn配置](#cdn配置)
- [html配置](#html配置)


`package.json` 如下，

```json
{
  "name": "vue2",
  "version": "1.0.0",
  "author": "",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "build": "node build/build.js"
  },
  "dependencies": {
    "@babel/polyfill": "^7.12.1"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "autodll-webpack-plugin": "^0.4.2",
    "autoprefixer": "^6.7.7",
    "babel-loader": "^8.2.1",
    "babel-preset-stage-2": "^6.24.1",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^3.5.3",
    "file-loader": "^6.0.0",
    "html-webpack-externals-plugin": "^3.8.0",
    "html-webpack-plugin": "^4.0.0-beta.14",
    "mini-css-extract-plugin": "^0.9.0",
    "optimize-css-assets-webpack-plugin": "^5.0.4",
    "postcss-loader": "^3.0.0",
    "px2rem-loader": "^0.1.9",
    "sass": "^1.29.0",
    "sass-loader": "^10.1.0",
    "style-loader": "^2.0.0",
    "url-loader": "^4.1.1",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.11.0"
  },
  "keywords": [
    "webpack",
    "scaffold"
  ],
  "license": "MIT"
}
```

### polyfill

兼容低版本浏览器
```
module.exports = {
  entry: ['@babel/polyfill', path.resolve(__dirname, '../src/index.js')] // webpack入口配置es6+语法兼容低版本浏览器
};
```

### babel

负责转码

- `@babel/core`

将源码转换成 `{ code, map, ast }` 结构
```
import * as babel from "@babel/core";

babel.transform(code, options, function(err, result) {
  result; // => { code, map, ast }
});



babel.transform("code();", options, function(err, result) {
  result.code;
  result.map;
  result.ast;
});
```


- `@babel/plugin-transform-runtime`

The transformation plugin is typically used only in development, but the runtime itself will be depended on by your deployed code

只在开发环境生效

一般配置在 `babel.config.js` 里
```js
module.exports = api => {
  return {
    plugins: ['@babel/plugin-transform-runtime'],
    presets: [
      [
        '@babel/preset-env',
        {
          useBuiltIns: 'entry',
          // 根据配置的浏览器兼容，引入浏览器不兼容的 polyfill。
          // 需要在入口文件手动添加 import '@babel/polyfill'，会自动根据 browserslist 替换成浏览器不兼容的所有 polyfill。
          // caller.target will be the same as the target option from webpack
          targets: api.caller(caller => caller && caller.target === 'node')
            ? { node: 'current' }
            : { chrome: '58', ie: '9' }
        },
        '@babel/preset-stage-2'
      ]
    ]
  };
};

```


- `@babel/preset-env`

转码到指定的版本，以便于更广泛的浏览器支持，比如，支持到 `ie9`


- `babel-loader`

加载 `js` 文件

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
            // plugins: ['@babel/transform-runtime']
          }
        },
        exclude: /node_modules/
      }
    ]
  }
}
```




### 打包优化

- `autodll-webpack-plugin`

能把第三方依赖的文件能提前进行预编译打包到一个文件里面去。提高构建速度

```
module.exports = {
  plugins: [
    new AutoDllPlugin({
    	inject: true, // will inject the DLL bundle to index.html
    	debug: true,
    	filename: '[name]_[hash].js',
    	path: './dll',
    	entry: {
    	  vendor: ['vue', 'vue-router']
    	}
    })
  ]
};
```


### 样式配置

- `autoprefixer`

添加样式前缀

`postcss.config.js`

```js
module.exports = {
  plugins: [require('autoprefixer')]
};

```



- `mini-css-extract-plugin`

提取文件里的样式代码，将CSS提取为独立的文件的插件，支持按需加载css和sourceMap


>在 `vue` 模板中的 `<style lang='scss'>` 使用 `scss`


```js
module.exports = {
  rules: [
    {
      test: /\.css$/,
      use: [
        {
          loader: MiniCssExtractPlugin.loader
        },
        {
          loader: 'css-loader'
        },
        {
          loader: 'postcss-loader',
          options: {
            plugins: [require('autoprefixer')]
          }
        },
        {
          loader: 'px2rem-loader',
          // options here
          options: {
            remUnit: 100, // 1rem = 100px
            remPrecision: 8 // 计算出rem的小数点的个数
          }
        }
      ]
    },
    {
      test: /(\.sass$)|(\.scss$)/,
      use: [
        {
          // loader: "style-loader" // creates style nodes from JS strings
          loader: MiniCssExtractPlugin.loader
        },
        {
          loader: 'css-loader' // translates CSS into CommonJS
        },
        {
          loader: 'postcss-loader',
          options: {
            plugins: [require('autoprefixer')]
          }
        },
        {
          loader: 'sass-loader' // compiles Less to CSS
        },
        {
          loader: 'px2rem-loader',
          // options here
          options: {
            remUnit: 100, // 1rem = 100px
            remPrecision: 8 // 计算出rem的小数点的个数
          }
        }
      ]
    }
  ]
}

```


需要安装 `yarn add px2rem-loader sass sass-loader style-loader -D`


- `optimize-css-assets-webpack-plugin`

压缩样式文件



- `postcss-loader`

加载样式文件


- `px2rem-loader`

样式不同屏幕尺寸兼容



### 文件清理

- `clean-webpack-plugin`

清理打包文件


- `css-loader`

会 `import/require()` 后再解析(`resolve`) `@import` 和 `url()` 语句

因此，通常会使用 `file-loader` 和 `url-loader`


### cdn配置

- `html-webpack-externals-plugin`

公司里建议搭建 `cdn` 服务

在 `webpack.config.js` 里配置

```js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackExternalsPlugin({
      externals: [
        {
          module: 'vue',
          entry: 'https://cdn.jsdelivr.net/npm/vue@2.6.11/dist/vue.min.js',
          global: 'Vue',
        },
        {
          module: 'react',
          entry: 'https://unpkg.com/react@17.0.1/umd/react.development.js',
          global: 'React',
        },
        {
          module: 'react-dom',
          entry: 'https://unpkg.com/react-dom@17/umd/react-dom.development.js',
          global: 'ReactDom',
        }
      ]
    })
  ]
};
```


### html配置

- `html-webpack-plugin`

```js
var HtmlWebpackPlugin = require('html-webpack-plugin');
var path = require('path');

var webpackConfig = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js'
  },
  plugins: [new HtmlWebpackPlugin()]
};
```

将打包后的代码注入到 html 里
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>webpack App</title>
  </head>
  <body>
    <script src="index_bundle.js"></script>
  </body>
</html>
```


## 项目配置

- [通用](#通用)
- [vue](#vue)
- [react](#react)


### 通用

```
yarn add eslint eslint-config-prettier eslint-config-standard eslint-plugin-import eslint-plugin-node eslint-plugin-prettier eslint-plugin-promise eslint-plugin-standard eslint-webpack-plugin -D
```

### vue

```
yarn add @vue/eslint-config-prettier eslint-config-vue eslint-plugin-vue -D
```


## 完整项目配置


- 配置结构
```
|- build/
|-   build.js
|-   webpack.base.conf.js
|-   webpack.dev.conf.js
|-   webpack.prod.conf.js
|- src/
|-   App.vue
|-   index.js
```
- [打包配置](#打包配置)
- [源码结构](#源码结构)


### 打包配置

- `build.js`

```js
const webpack = require('webpack');
const config = require('./webpack.prod.conf');

webpack(config, (err, stats) => {
  if (err || stats.hasErrors()) {
    // 处理错误
    console.error(err);
    return;
  }
  console.log(
    stats.toString({
      chunks: false, // 使构建过程静默无输出
      colors: true // 在控制台输出颜色字体
    })
  );
});

```


- `webpack.base.conf.js` 文件

```js
const webpack = require('webpack');
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const AutoDllPlugin = require('autodll-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
// const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

module.exports = {
  entry: ['@babel/polyfill', path.resolve(__dirname, '../src/index.js')],
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js'
  },
  resolve: {
    extensions: ['*', '.js', '.json', '.vue'],
    alias: {
      vue$: 'vue/dist/vue.esm.js',
      '@': path.resolve(__dirname, '../src')
    }
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        include: [path.resolve(__dirname, '../src')]
      },
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
            // plugins: ['@babel/transform-runtime']
          }
        },
        exclude: /node_modules/
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[path][name].[ext]?[hash]'
            }
          }
        ]
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[path][name].[ext]?[hash]'
            }
          }
        ]
      },
      {
        test: /\.(mp3|wav|wma|ape|aac)$/i,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[path][name].[ext]?[hash]'
            }
          }
        ]
      },
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          {
            loader: 'css-loader'
          },
          {
            loader: 'postcss-loader',
            options: {
              plugins: [require('autoprefixer')]
            }
          },
          {
            loader: 'px2rem-loader',
            // options here
            options: {
              remUnit: 100, // 1rem = 100px
              remPrecision: 8 // 计算出rem的小数点的个数
            }
          }
        ]
      },
      {
        test: /(\.sass$)|(\.scss$)/,
        use: [
          {
            // loader: "style-loader" // creates style nodes from JS strings
            loader: MiniCssExtractPlugin.loader
          },
          {
            loader: 'css-loader' // translates CSS into CommonJS
          },
          {
            loader: 'postcss-loader',
            options: {
              plugins: [require('autoprefixer')]
            }
          },
          {
            loader: 'sass-loader' // compiles Less to CSS
          },
          {
            loader: 'px2rem-loader',
            // options here
            options: {
              remUnit: 100, // 1rem = 100px
              remPrecision: 8 // 计算出rem的小数点的个数
            }
          }
        ]
      }
    ]
  },
  optimization: {
    splitChunks: {
      // minSize: 0, // 包最小的体积 0代表无条件提取
      cacheGroups: {
        // commons:{ // 抽取基础库 vue、vue-router、vuex、react
        //   test:/(vue|vue-router|vuex)/,
        //   name:'vendors',
        //   chunks: 'all'// 同步&异步加载的都提取出来
        // },
        commons: {
          // 抽取基础库 vue、vue-router、vuex、react
          name: 'commons',
          chunks: 'all', // 同步&异步加载的都提取出来
          minChunks: 2
        }
      }
    }
  },
  plugins: [
    new MiniCssExtractPlugin(),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../index.html')
    }),
    new VueLoaderPlugin(),
    new webpack.optimize.SplitChunksPlugin(),
    // 抽取基础库 vue、react
    new HtmlWebpackExternalsPlugin({
      externals: [
        {
          module: 'vue',
          entry: 'https://cdn.jsdelivr.net/npm/vue@2.6.11/dist/vue.min.js',
          global: 'Vue',
        },
        {
          module: 'react',
          entry: 'https://unpkg.com/react@17.0.1/umd/react.development.js',
          global: 'React',
        },
        {
          module: 'react-dom',
          entry: 'https://unpkg.com/react-dom@17/umd/react-dom.development.js',
          global: 'ReactDom',
        }
      ]
    })
    // Dll 优化打包速度
    // new AutoDllPlugin({
    // 	inject: true, // will inject the DLL bundle to index.html
    // 	debug: true,
    // 	filename: '[name]_[hash].js',
    // 	path: './dll',
    // 	entry: {
    // 	  vendor: ['vue', 'vue-router']
    // 	}
    // }),
  ]
};
```


- `webpack.dev.conf.js`

```js
const merge = require('webpack-merge');
const path = require('path');
const baseConfig = require('./webpack.base.conf');

module.exports = merge(baseConfig, {
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    contentBase: path.resolve(__dirname, '../dist'),
    open: true
  }
});

```

- `webpack.prod.conf.js`

```js
const merge = require('webpack-merge');
const path = require('path');
const baseConfig = require('./webpack.base.conf');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = merge(baseConfig, {
  mode: 'production',
  devtool: 'source-map',
  plugins: [
    new CleanWebpackPlugin({
      root: path.resolve(__dirname, '../'),
      verbose: true,
      dry: false
    }),
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /\.optimize\.css$/g,
      cssProcessor: require('cssnano'),
      cssProcessorPluginOptions: {
        preset: ['default', { discardComments: { removeAll: true } }]
      },
      canPrint: true
    })
  ]
});
```


### 源码结构

```
|- src/
|-   index.js
|-   App.vue
```

```js
import Vue from 'vue';
import App from './App';
import router from './router';

import './styles/index.scss';

new Vue({
  router: router,
  render: h => h(App)
}).$mount('#app');

```

`App.vue`
```vue
<template>
  <div class="app">
    <router-view></router-view>
    <!-- 删除试试 底部菜单 -->
    <FooterCmp></FooterCmp>
  </div>
</template>

<script>
import FooterCmp from './components/FooterCmp.vue';

export default {
  components: {
    FooterCmp
  }
};
</script>

```

