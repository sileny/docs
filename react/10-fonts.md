# 处理字体文件

> webpack.config.js
```ecmascript 6
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  module: { // 处理类型文件
    rules: [
      {test: /\.jsx?$/, use: 'babel-loader', exclude: /node_modules/},
      {
        test: /\.s?css?$/,
        // use: ['style-loader', 'sass-loader', 'css-loader?modules'/* 表示启用模块化 */],
        use: [
          'style-loader',
          'sass-loader',
          'css-loader?modules&localIdentName=[path][name]-[local]-[hash:8]'
        ],
        exclude: /node_modules/
      },
      {test: /\.(woff2?|eot|ttf|otf|mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/i, use: 'url-loader'},
      {test: /\\.(svg)(\?.*)?$/, use: 'file-loader'}
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 'src/index.html', // 源文件模板
      filename: 'index.html' // 会被打包到内存中(访问 localhost:3000/index.html)
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
