# 默认不支持jsx

修改 webpack.config.js
```ecmascript 6
// webpack.config.js

const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  module: { // 处理类型文件
    rules: [
      {test: /\.jsx?$/, use: 'babel-loader', exclude: /node_modules/}
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 'src/index.html', // 源文件模板
      filename: 'index.html' // 会被打包到内存中
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