# 脚手架基本配置


- [基础配置](#基础配置)
- [项目规范配置](#项目规范配置)
- [项目配置](#项目配置)
- [环境变量配置](#环境变量配置)
- [热加载](#热加载)


## 基础配置

- [package.json](#package)
- [polyfill](#polyfill)
- [babel](#babel)
- [打包优化](#打包优化)
- [样式配置](#样式配置)
- [文件清理](#文件清理)
- [cdn配置](#cdn配置)
- [html配置](#html配置)


### package

`package.json` 如下，

```json
{
  "name": "webpack-template",
  "version": "1.0.0",
  "author": "",
  "description": "webpack template",
  "main": "index.js",
  "scripts": {
    "analyze": "webpack --env production --config webpack-analyze.js && webpack-bundle-analyzer dist/stats.json",
    "dev": "webpack-dev-server --env development --inline --progress --config webpack-dev.js",
    "build": "webpack --env production --config webpack-prod.js"
  },
  "dependencies": {
    "@babel/polyfill": "^7.12.1",
    "@silen/font-size": "^1.1.0",
    "normalize.css": "^8.0.1"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "autoprefixer": "^6.7.7",
    "babel-eslint": "^10.1.0",
    "babel-loader": "^8.2.1",
    "babel-preset-stage-2": "^6.24.1",
    "clean-webpack-plugin": "^3.0.0",
    "compression-webpack-plugin": "^6.1.1",
    "css-loader": "^3.5.3",
    "eslint": "^7.13.0",
    "eslint-config-prettier": "^6.15.0",
    "eslint-config-standard": "^16.0.1",
    "eslint-plugin-import": "^2.22.1",
    "eslint-plugin-node": "^11.1.0",
    "eslint-plugin-prettier": "^3.1.4",
    "eslint-plugin-promise": "^4.2.1",
    "eslint-plugin-standard": "^4.1.0",
    "eslint-webpack-plugin": "^2.3.0",
    "file-loader": "^6.0.0",
    "html-webpack-externals-plugin": "^3.8.0",
    "html-webpack-plugin": "^4.5.0",
    "husky": "^4.3.0",
    "lint-staged": "^10.5.1",
    "mini-css-extract-plugin": "^0.9.0",
    "optimize-css-assets-webpack-plugin": "^5.0.4",
    "postcss-loader": "^3.0.0",
    "prettier": "^2.1.2",
    "px2rem-loader": "^0.1.9",
    "sass": "^1.29.0",
    "sass-loader": "^10.1.0",
    "style-loader": "^2.0.0",
    "url-loader": "^4.1.1",
    "webpack": "^5.6.0",
    "webpack-bundle-analyzer": "^4.1.0",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.4.0"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/sileny/webpack-template.git"
  },
  "keywords": [
    "webpack5",
    "vue",
    "react"
  ],
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
    "src/**/*.{css,scss,less,html,md,markdown}": [
      "prettier --write"
    ]
  },
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

转码到指定的版本，以便于更广泛的浏览器支持，比如，上面的配置支持到 `ie9`


**注意**

如果是 `vue`，需要安装 `@vue/babel-preset-app`，本项目安装的是 `"@vue/babel-preset-app": "^4.5.8",`


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

但推荐使用更好的 [hard-source-webpack-plugin](https://github.com/mzgoddard/hard-source-webpack-plugin)



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

```js
module.exports = {
  rules: [
    {
      test: /.(sa|sc|c)ss$/,
      use: [
        { loader: 'css-loader' },
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
        },
        // Uncaught Error: Module build failed (from ./node_modules/px2rem-loader/index.js):
        // Error: undefined:21:5: property missing ':'
        { loader: 'sass-loader' }
      ]
    }
  ]
}

```

需要安装 `yarn add px2rem-loader sass sass-loader style-loader -D`


**注意**

1. 在 `vue` 模板中的 `<style lang='scss'>` 使用 `scss` 或者别的样式语言，需要使用 `vue-style-loader`
```js
module.exports = {
  rules: [
    {
      test: /.(sa|sc|c)ss$/,
      use: [
        { loader: 'vue-style-loader' },
        { loader: 'css-loader' }
      ]
    }
  ]
};
```

2. `px2rem-loader` 需要在 __预编译样式语言__ 之前，否则，会报错



- `optimize-css-assets-webpack-plugin`

压缩样式文件


- `css-loader`

会 `import/require()` 后再解析(`resolve`) `@import` 和 `url()` 语句

因此，通常会使用 `file-loader` 和 `url-loader`



- `postcss-loader`

加载样式文件


- `px2rem-loader`

样式不同屏幕尺寸兼容



### 文件清理

- `clean-webpack-plugin`

清理打包文件


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

引入外部资源后，需要注释 `import` 及 `Vue.use(xxx)`，此时，资源就完整地加载到内存，就没有按需引入这个说法
```js
// import Vue from 'vue'
// import ElementUI from 'element-ui'
// import 'element-ui/lib/theme-chalk/index.css'
import App from './App'
 
Vue.config.productionTip = false
 
// Vue.use(ElementUI)
```

**注**

如果项目里配置了 `eslint`，那么，防止 `eslint` 报错，可以在 `globals` 里做配置
```
globals: {
  Vue: false
},
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


## 项目规范配置

- [通用](#通用)
- [代码规范](#代码规范)
- [vue](#vue)
- [react](#react)


### 通用

```
yarn add eslint eslint-config-prettier eslint-config-standard eslint-plugin-import eslint-plugin-node eslint-plugin-prettier eslint-plugin-promise eslint-plugin-standard eslint-webpack-plugin -D
```

### 代码规范

首先，要初始化仓库

```
git init
```

因为用到了 `husky`，后安装，会导致 `husky` 提交验证的时候失效

```
yarn add husky lint-staged prettier -D
```

在 `package.json` 里添加如下配置
```json
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


### vue

```
yarn add @vue/eslint-config-prettier eslint-config-vue eslint-plugin-vue -D
```


>`vue-loader`、`vue-style-loader`、`vue-template-compiler`，其中，`vue-template-compiler` 要和 `vue` 版本保持一致


此时的 `babel.config.js`

```js
module.exports = api => {
  return {
    plugins: ['@babel/plugin-transform-runtime'],
    presets: [
      '@vue/app', // yarn add @vue/babel-preset-app -D
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


### react

```
yarn add react react-router-dom redux react-redux
```



## 项目配置


- 配置结构
```
|- src/
|-   App.vue
|-   index.js
|- webpack-analyze.js
|- webpack-base.js
|- webpack-dev.js
|- webpack-prod.js
```
- [打包配置](#打包配置)
- [源码结构](#源码结构)


### 打包配置

- `webpack-base.js`

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: ['@babel/polyfill', path.resolve(__dirname, './src/index.js')],
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'js/[name].js'
  },
  resolve: {
    extensions: ['.js', '.json', '.scss', '.less'],
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  },
  module: {
    rules: [
      {
        test: /.js$/,
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
        test: /.(png|svg|jpg|gif)$/,
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
        test: /.(woff|woff2|eot|ttf|otf)$/,
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
        test: /.(mp3|wav|wma|ape|aac)$/i,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[path][name].[ext]?[hash]'
            }
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, './index.html')
    })
  ]
};

```

**注意**

如果是编译 `vue` 项目，需要用到 `vue-loader` 加载 `vue` 文件，样式编译，用到 `vue-style-loader`
```js
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  resolve: {
    extensions: ['.js', '.json', '.vue', '.scss', '.less'],
    alias: {
      vue$: 'vue/dist/vue.esm.js',
      '@': path.resolve(__dirname, './src')
    }
  },
  module: {
    rules: [
      {
        test: /.vue$/,
        use: ['vue-loader'],
        include: [path.resolve(__dirname, './src')]
      },
      {
        test: /.(sa|sc|c)ss$/,
        use: [
          { loader: 'vue-style-loader' },
          { loader: 'css-loader' },
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
          },
          { loader: 'sass-loader' }
        ]
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
};
```


- `webpack-dev.js` 文件

```js
const path = require('path');
const webpack = require('webpack');
const { merge } = require('webpack-merge');

const base = require('./webpack-base');

module.exports = (env, args) => {
  const { host, port, BRANCH } = args;

  return merge(base, {
    mode: env,
    devtool: 'inline-source-map',
    devServer: {
      compress: true,
      contentBase: path.resolve(__dirname, './dist'),
      hot: true,
      open: true
    },
    entry: [`webpack-dev-server/client?http://${host}:${port || 8080}/`], // 热加载配置，支持命令行传参：--host 127.0.0.1 --port 12345
    module: {
      rules: [
        {
          test: /.(sa|sc|c)ss$/,
          use: [
            { loader: 'css-loader' },
            {
              loader: 'postcss-loader',
              options: {
                plugins: [require('autoprefixer')] // 前缀
              }
            },
            {
              loader: 'px2rem-loader',
              // options here
              options: {
                remUnit: 100, // 1rem = 100px
                remPrecision: 8 // 计算出rem的小数点的个数
              }
            },
            // Uncaught Error: Module build failed (from ./node_modules/px2rem-loader/index.js):
            // Error: undefined:21:5: property missing ':'
            { loader: 'sass-loader' }
          ]
        }
      ]
    },
    optimization: {
      splitChunks: {
        minSize: 0, // 包最小的体积 0代表无条件提取
        chunks: 'async',
        maxSize: 20000, // 为0，则每个文件在每个阶段被编译后的文件内容都可以被访问到，例如，http://localhost:8080/js/main~src_pages_index_vue_这里的哈希不同而已.js
        minChunks: 1,
        maxAsyncRequests: 30,
        maxInitialRequests: 30,
        automaticNameDelimiter: '~',
        cacheGroups: {
          commons: {
            // 抽取基础库 vue、vue-router、vuex、react、react-dom
            name: 'commons',
            chunks: 'all', // 同步&异步加载的都提取出来
            minChunks: 2
          }
        }
      }
    },
    plugins: [
      new webpack.DefinePlugin({ // 注入环境变量
        'process.env': JSON.stringify({
          BRANCH,
          TIME: Date.now(),
          VERSION: JSON.stringify('5fa3b9'),
          BROWSER_SUPPORTS_HTML5: true
        })
      }),
      new webpack.HotModuleReplacementPlugin() // 支持热加载
    ]
  });
};

```

热加载配置看[这里](#热加载)


- `webpack-prod.js`

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CompressionWebpackPlugin = require('compression-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const webpack = require('webpack');
const { merge } = require('webpack-merge');

const base = require('./webpack-base');

module.exports = (env, args) => {
  const { BRANCH } = args;

  return merge(base, {
    mode: env,
    devtool: 'source-map',
    module: {
      rules: [
        {
          test: /.(sa|sc|c)ss$/,
          use: [
            { loader: MiniCssExtractPlugin.loader }, // 打包提取样式需要将 `style-loader` 提取出来的 `style` 放在 `.css` 文件里
            { loader: 'css-loader' },
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
            },
            // Uncaught Error: Module build failed (from ./node_modules/px2rem-loader/index.js):
            // Error: undefined:21:5: property missing ':'
            { loader: 'sass-loader' }
          ]
        }
      ]
    },
    optimization: {
      splitChunks: {
        cacheGroups: {
          commons: {
            // 抽取基础库 vue、vue-router、vuex、react、react-dom
            test: /(vue|vue-router|vuex)/,
            name: 'commons',
            chunks: 'all' // 同步&异步加载的都提取出来
          }
        }
      }
    },
    plugins: [
      new CleanWebpackPlugin({
        root: path.resolve(__dirname, './'),
        verbose: true,
        dry: false
      }),
      new webpack.DefinePlugin({
        'process.env': JSON.stringify({
          BRANCH,
          TIME: Date.now(),
          VERSION: JSON.stringify('5fa3b9'),
          BROWSER_SUPPORTS_HTML5: true
        })
      }),
      new CompressionWebpackPlugin({
        filename: '[file].gz[query]',
        algorithm: 'gzip',
        threshold: 10240,
        test: /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i,
        minRatio: 0.8,
        deleteOriginalAssets: false
      }),
      new OptimizeCssAssetsPlugin({
        assetNameRegExp: /.css$/g,
        cssProcessor: require('cssnano'),
        cssProcessorPluginOptions: {
          preset: ['default', { discardComments: { removeAll: true } }]
        },
        canPrint: true
      }),
      new MiniCssExtractPlugin({
        filename: 'css/[name].css',
        chunkFilename: 'css/[id].css'
      })
    ]
  });
};

```



- `webpack-analyze.js`

分析项目包
```js
const { merge } = require('webpack-merge');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

const prod = require('./webpack-prod');

module.exports = (env, args) => {
  return merge(prod(env, args), {
    plugins: [
      new BundleAnalyzerPlugin({
        analyzerMode: 'disabled',
        generateStatsFile: true,
        statsOptions: { source: false }
      })
    ]
  });
};

```


### scripts

项目使用了多个配置文件，其执行脚本如下，
```
"scripts": {
  "analyze": "webpack --env production --config webpack-analyze.js && webpack-bundle-analyzer dist/stats.json",
  "dev": "webpack-dev-server --env development --inline --progress --config webpack-dev.js",
  "build": "webpack --env production --config webpack-prod.js"
},
```

简化的配置在[这里](#最终配置)



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

完整的项目在[这里](https://github.com/sileny/webpack-template)


## 环境变量配置


本项目案例使用的是 `webpack@^5.6.0`，支持 `DefinePlugin`，如果部分 `webpack` 不支持，那么，可以使用下面的方式进行配置



浏览器环境下读取到环境变量的值，靠的是 `mode` ，在 node 环境下读取到环境变量的值，靠的是 `cross-env`

- `DefinePlugin` 的作用：是设置浏览器环境下能读取到的 `全局变量`，直接通过 `key` 读取，在 `node` 环境下是无法读取到的
- `cross-env` 的作用：是通过命令行设置环境变量 `NODE_ENV`，使 `node` 环境下能读取到，通过 `process.env.NODE_ENV` 读取
- 如果在 `DefinePlugin` 里设置的 `key` 是 `process.env.NODE_ENV` ，会覆盖 `webpack` 通过 `mode` 模式设置的环境变量的值


安装依赖
```
yarn add cross-env -D
```

在 `package.json` 添加配置
```
"scripts": {
  "dev": "cross-env NODE_ENV='development' webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
  "build": "cross-env NODE_ENV='production' node build/build.js"
},
```

下面是将配置都写在 `webpack.config.js` 里之后做的脚本配置
```
"dev": "cross-env NODE_ENV='development' webpack-dev-server --env development --inline --progress",
"build": "cross-env NODE_ENV='production' webpack --env production --mode=production"
```
比下面的配置少 `0.6k` 的代码，应该是在为嵌入 `if (process.env.NODE_ENV)` 之类的代码
```
"dev": "webpack-dev-server --env development --inline --progress",
"build": "webpack --env production --mode=production"
```


## 热加载

开发模式为了看到实时地效果，配置热加载，可以大大提升效率
```js
module.exports = {
  entry: [
    '@babel/polyfill',                                  // 兼容性
    path.resolve(__dirname, '../src/index.js'),         // 程序入口
    'webpack-dev-server/client?http://localhost:8080/'  // 热加载的关键
  ],
  devServer: {
    compress: true,                                     // gzip压缩
    contentBase: path.resolve(__dirname, '../dist'),    // contentbase代表html页面所在的相对目录，如果不配置项，devServer默认html所在的目录就是项目的根目录
    hot: true,                                          // 启用热加载
    open: true                                          // 自动打开浏览器
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()            // 热加载插件
  ]
}
```

如果只是配置了插件，即 `new webpack.HotModuleReplacementPlugin()`，是不会实现热加载的，只会输出以下控制台日志
```
[HMR] Waiting for update signal from WDS...
```


热加载配置成功后，控制台会有如下日志输出，
```
[HMR] Waiting for update signal from WDS...
[WDS] Hot Module Replacement enabled.
[WDS] Live Reloading enabled.
```

当改变文件后，会有以下日志输出
```
[WDS] 100% - Compilation completed.
[HMR] Updated modules:
[HMR]  - ./src/pages/index.vue?vue&type=script&lang=js&
[HMR]  - ./src/pages/index.vue?vue&type=script&lang=js&
[HMR]  - ./src/pages/index.vue
[HMR]  - ./src/pages/index.vue?vue&type=template&id=57509004&
[HMR]  - ./src/pages/index.vue?vue&type=template&id=57509004&
[HMR] App is up to date.
```


## 最终配置

- `webpack.config.js`
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');
const CompressionWebpackPlugin = require('compression-webpack-plugin');
const webpack = require('webpack');

module.exports = (env, args) => {
  const { host, port, BRANCH, analyze } = args;
  const isProd = env === 'production';
  const entry = ['@babel/polyfill', path.resolve(__dirname, './src/index.js')];
  let devServer = {};
  let devtool = 'source-map';
  const plugins = [];
  let optimization = {
    splitChunks: {
      minSize: 0, // 包最小的体积 0代表无条件提取
      chunks: 'async',
      maxSize: 20000, // 为0，则每个文件在每个阶段被编译后的文件内容都可以被访问到，例如，http://localhost:8080/js/main~src_pages_index_vue_这里的哈希不同而已.js
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      automaticNameDelimiter: '~',
      cacheGroups: {
        // commons:{ // 抽取基础库 vue、vue-router、vuex、react、react-dom
        //   test:/(vue|vue-router|vuex)/,
        //   name:'vendors',
        //   chunks: 'all'// 同步&异步加载的都提取出来
        // },
        commons: {
          // 抽取基础库 vue、vue-router、vuex、react、react-dom
          name: 'commons',
          chunks: 'all', // 同步&异步加载的都提取出来
          minChunks: 2
        }
      }
    }
  };

  if (!isProd) {
    entry.push(`webpack-dev-server/client?http://${host}:${port}/`);

    devServer = {
      compress: true,
      contentBase: path.resolve(__dirname, './dist'),
      hot: true,
      open: true
    };

    devtool = 'inline-source-map';

    plugins.push(new webpack.HotModuleReplacementPlugin());
  } else {
    const { CleanWebpackPlugin } = require('clean-webpack-plugin');
    const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
    plugins.push(
      new CleanWebpackPlugin({
        root: path.resolve(__dirname, './'),
        verbose: true,
        dry: false
      }),
      new OptimizeCssAssetsPlugin({
        assetNameRegExp: /.css$/g,
        cssProcessor: require('cssnano'),
        cssProcessorPluginOptions: {
          preset: ['default', { discardComments: { removeAll: true } }]
        },
        canPrint: true
      })
    );
    optimization = {
      splitChunks: {
        cacheGroups: {
          commons: {
            // 抽取基础库 vue、vue-router、vuex、react、react-dom
            test: /(vue|vue-router|vuex)/,
            name: 'commons',
            chunks: 'all' // 同步&异步加载的都提取出来
          }
        }
      }
    };
    if (analyze) {
      plugins.push(
        new (require('webpack-bundle-analyzer').BundleAnalyzerPlugin)({
          analyzerMode: 'disabled',
          generateStatsFile: true,
          statsOptions: { source: false }
        })
      );
    }
  }

  return {
    mode: env,
    devtool,
    devServer,
    entry,
    optimization,
    output: {
      path: path.resolve(__dirname, './dist'),
      filename: 'js/[name].js'
    },
    resolve: {
      extensions: ['.js', '.json', '.vue', '.scss', '.less'],
      alias: {
        vue$: 'vue/dist/vue.esm.js',
        '@': path.resolve(__dirname, './src')
      }
    },
    module: {
      rules: [
        {
          test: /.vue$/,
          use: ['vue-loader'],
          include: [path.resolve(__dirname, './src')]
        },
        {
          test: /.js$/,
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
          test: /.(png|svg|jpg|gif)$/,
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
          test: /.(woff|woff2|eot|ttf|otf)$/,
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
          test: /.(mp3|wav|wma|ape|aac)$/i,
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
          test: /.(sa|sc|c)ss$/,
          use: [
            { loader: isProd ? MiniCssExtractPlugin.loader : 'vue-style-loader' },
            { loader: 'css-loader' },
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
            },
            // Uncaught Error: Module build failed (from ./node_modules/px2rem-loader/index.js):
            // Error: undefined:21:5: property missing ':'
            { loader: 'sass-loader' }
          ]
        }
      ]
    },
    plugins: [
      ...plugins,
      new webpack.DefinePlugin({
        'process.env': JSON.stringify({
          BRANCH,
          TIME: Date.now(),
          VERSION: JSON.stringify('5fa3b9'),
          BROWSER_SUPPORTS_HTML5: true
        })
      }),
      new CompressionWebpackPlugin({
        filename: '[file].gz[query]',
        algorithm: 'gzip',
        threshold: 10240,
        test: /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i,
        minRatio: 0.8,
        deleteOriginalAssets: false
      }),
      new MiniCssExtractPlugin({
        filename: 'css/' + (isProd ? '[name].css' : '[name].[contenthash].css'),
        chunkFilename: 'css/' + (isProd ? '[id].css' : '[id].[contenthash].css')
      }),
      new HtmlWebpackPlugin({
        template: path.resolve(__dirname, './index.html')
      }),
      new VueLoaderPlugin(),
      new HtmlWebpackExternalsPlugin({
        externals: [
          {
            module: 'vue',
            entry: 'https://unpkg.com/vue@2.6.11/dist/vue.min.js',
            global: 'Vue'
          },
          {
            module: 'vue-router',
            entry: 'https://unpkg.com/vue-router@3.2.0/dist/vue-router.min.js',
            global: 'VueRouter'
          },
          {
            module: 'vuex',
            entry: 'https://unpkg.com/vuex@3.5.1/dist/vuex.min.js',
            global: 'Vuex'
          }
        ]
      })
    ]
  };
};

```

**Vue项目配置**，参考[打包配置`webpack-base.js`](#打包配置)


- `scripts`
```
"scripts": {
  "analyze": "webpack --env production --analyze && webpack-bundle-analyzer dist/stats.json",
  "dev": "webpack-dev-server --env development --inline --progress",
  "build": "webpack --env production"
},
```

- 完整的 `package.json` 配置

```
{
  "name": "webpack-template",
  "version": "1.0.0",
  "author": "",
  "description": "",
  "main": "index.js",
  "scripts": {
    "analyze": "webpack --env production --analyze && webpack-bundle-analyzer dist/stats.json",
    "dev": "webpack-dev-server --env development --inline --progress",
    "build": "webpack --env production"
  },
  "dependencies": {
    "@babel/polyfill": "^7.12.1",
    "@silen/font-size": "^1.1.0",
    "normalize.css": "^8.0.1",
    "vue": "^2.6.11",
    "vue-router": "^3.2.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "@vue/babel-preset-app": "^4.5.8",
    "@vue/eslint-config-prettier": "^6.0.0",
    "autoprefixer": "^6.7.7",
    "babel-eslint": "^10.1.0",
    "babel-loader": "^8.2.1",
    "babel-preset-stage-2": "^6.24.1",
    "clean-webpack-plugin": "^3.0.0",
    "compression-webpack-plugin": "^6.1.1",
    "css-loader": "^3.5.3",
    "eslint": "^7.13.0",
    "eslint-config-prettier": "^6.15.0",
    "eslint-config-standard": "^16.0.1",
    "eslint-config-vue": "^2.0.2",
    "eslint-plugin-import": "^2.22.1",
    "eslint-plugin-node": "^11.1.0",
    "eslint-plugin-prettier": "^3.1.4",
    "eslint-plugin-promise": "^4.2.1",
    "eslint-plugin-standard": "^4.1.0",
    "eslint-plugin-vue": "^7.1.0",
    "eslint-webpack-plugin": "^2.3.0",
    "file-loader": "^6.0.0",
    "html-webpack-externals-plugin": "^3.8.0",
    "html-webpack-plugin": "^4.5.0",
    "husky": "^4.3.0",
    "lint-staged": "^10.5.1",
    "mini-css-extract-plugin": "^0.9.0",
    "optimize-css-assets-webpack-plugin": "^5.0.4",
    "postcss-loader": "^3.0.0",
    "prettier": "^2.1.2",
    "px2rem-loader": "^0.1.9",
    "sass": "^1.29.0",
    "sass-loader": "^10.1.0",
    "style-loader": "^2.0.0",
    "url-loader": "^4.1.1",
    "vue-loader": "^15.9.2",
    "vue-style-loader": "^4.1.2",
    "vue-template-compiler": "^2.6.11",
    "webpack": "^5.6.0",
    "webpack-bundle-analyzer": "^4.1.0",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.4.0"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/sileny/webpack-template.git"
  },
  "keywords": [
    "webpack5",
    "vue",
    "react"
  ],
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,vue,json}": [
      "prettier --write",
      "eslint --fix"
    ],
    "src/**/*.{css,scss,less,html,md,markdown}": [
      "prettier --write"
    ]
  },
  "license": "MIT"
}

```

