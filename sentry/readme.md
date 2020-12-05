# sentry

遇到自己和测试的环境都OK， 客户使用有各种各样还复现不了的问题，因此可以使用：前端异常日志监控!

- [vue](#vue)


# vue

如果在组件渲染时出现运行错误，错误将会被传递至全局 Vue.config.errorHandler 配置函数 (如果已设置)。利用这个钩子函数来配合错误跟踪服务是个不错的主意。比如 Sentry，它为 Vue 提供了[官方集成](https://cn.vuejs.org/v2/guide/deployment.html#%E8%B7%9F%E8%B8%AA%E8%BF%90%E8%A1%8C%E6%97%B6%E9%94%99%E8%AF%AF)。


## 注册及创建项目

注册地址 `https://sentry.io/auth/login/`

选择 `vue`，创建项目

## 配置你的vue项目

1、Install the dependencies:

```
yarn add @sentry/vue @sentry/tracing

npm install --save @sentry/vue @sentry/tracing
```

Next, initialize Sentry in your app entry point before you initialize your root component.


2、`main.js`

```js
import * as Sentry from '@sentry/browser';
import { Integrations } from '@sentry/tracing';

Sentry.init({
  Vue,
  dsn: 'https://24aa4a2627734e218c200c25232de389@o486490.ingest.sentry.io/5543839',
  integrations: [
    new Integrations.BrowserTracing()
  ],

  // We recommend adjusting this value in production, or using tracesSampler
  // for finer control
  tracesSampleRate: 1.0,
  logErrors: true
});
```

开发环境的控制台没有报错信息了， 只需要配置： `logErrors: true`

>可能需要一个版本信息， 以便确定问题是哪个版本的问题，例如：`release: test@1.0.1`

如果开发环境完全不需要上传日志啊，那就加个判断

```js
process.env.NODE_ENV === "production" && Sentry.init({
  Vue,
  dsn: 'https://24aa4a2627734e218c200c25232de389@o486490.ingest.sentry.io/5543839',
  integrations: [
    new Integrations.BrowserTracing()
  ],

  // We recommend adjusting this value in production, or using tracesSampler
  // for finer control
  tracesSampleRate: 1.0,
  logErrors: true
});
```


3、Map文件

安装 `@sentry/webpack-plugin`

```
yarn add @sentry/webpack-plugin -D

// or
npm i @sentry/webpack-plugin -D
```




创建一个 `.sentryclirc` 文件，配置服务监控，内容如下
```
[auth]
token=2daf1a3f5312399ab075fd7fa7178b828ad6e1f0ee31431c9edf7b1a9793679e

[defaults]
url=https://sentry.io
org=组织名
project=项目名

```

- 组织名就是 `General Settings` 下 `Organization Slug` 对应的名字
- `Projects` 列表里的项目名就是这里指定的项目名

运行项目
```
 INFO  Starting development server...
> Found 98 release files
> Analyzing 98 sources
> Rewriting sources
> Adding source map references                                                                                                      8
> Bundled 98 files for upload                                                                                                       8
> Uploaded release files to Sentry                                                                                                  8
> File upload complete                                                                                                              )

Source Map Upload Report
  Scripts
    ~/js/md5.js
    ~/lib/bootstrap/js/bootstrap.js
    ~/lib/bootstrap/js/npm.js
  Minified Scripts
    ~/js/app.a4d855e4.js (sourcemap at app.a4d855e4.js.map)
    ~/js/chunk-0089a90d.53ec9e33.js (sourcemap at chunk-0089a90d.53ec9e33.js.map)
    ~/js/chunk-03859c52.74120004.js (sourcemap at chunk-03859c52.74120004.js.map)
    ~/js/chunk-03dacdbd.44cc61db.js (sourcemap at chunk-03dacdbd.44cc61db.js.map)
    ~/js/chunk-0916bcbe.bc93bf21.js (sourcemap at chunk-0916bcbe.bc93bf21.js.map)
    ~/js/chunk-0e13ace5.2e79b2d6.js (sourcemap at chunk-0e13ace5.2e79b2d6.js.map)
    ~/js/chunk-vendors.e98f6f1a.js (sourcemap at chunk-vendors.e98f6f1a.js.map)
    ~/js/jquery.min.js (no sourcemap ref)
      - warning: could not determine a source map reference (Could not auto-detect referenced sourcemap for ~/js/jquery.min.js.)
    ~/js/vue.min.js (no sourcemap ref)
      - warning: could not determine a source map reference (Could not auto-detect referenced sourcemap for ~/js/vue.min.js.)
    ~/lib/bootstrap/js/bootstrap.min.js (no sourcemap ref)
      - warning: could not determine a source map reference (Could not auto-detect referenced sourcemap for ~/lib/bootstrap/js/bootstrap.min.js.)
    ~/lib/toastr/toastr.min.js (sourcemap at toastr.js.map)
      - warning: could not determine a source map reference (Could not auto-detect referenced sourcemap for ~/lib/toastr/toastr.min.js.)
    ~/socket.io.js (sourcemap at socket.io.js.map)
      - warning: could not determine a source map reference (Could not auto-detect referenced sourcemap for ~/socket.io.js.)
  Source Maps
    ~/js/app.a4d855e4.js.map
    ~/js/chunk-0089a90d.53ec9e33.js.map
    ~/js/chunk-03859c52.74120004.js.map
    ~/js/chunk-03dacdbd.44cc61db.js.map
    ~/js/chunk-0916bcbe.bc93bf21.js.map
    ~/js/chunk-0e13ace5.2e79b2d6.js.map
    ~/js/chunk-vendors.e98f6f1a.js.map
    ~/lib/bootstrap/css/bootstrap-theme.css.map
    ~/lib/bootstrap/css/bootstrap-theme.min.css.map
    ~/lib/bootstrap/css/bootstrap.css.map
    ~/lib/bootstrap/css/bootstrap.min.css.map


 DONE  Compiled successfully in 47904ms
```

