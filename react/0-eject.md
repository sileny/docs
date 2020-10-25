# 暴露react的项目配置

以下配置在老版本中需要做如下配置，在新版本里不需要
```
npm run eject
// 会释放`config`、`scripts`文件夹
```

这样，可以修改项目配置

这里修改成不需要运行`npm run "build-css"`直接支持`.scss`文件

- 安装 sass-loader
```
npm install sass-loader --save-dev
```

- 修改`config/webpack.config.dev.js`

```ecmascript 6
// 添加在 191 行
{test: /\.scss/, loaders: ['style-loader', 'css-loader', 'sass-loader']},
```
- 同时，也要修改 `config/webpack.config.prod.js`

```ecmascript 6
// 添加在 215 行
{test: /\.scss/, loaders: ['style-loader', 'css-loader', 'sass-loader']},
```
