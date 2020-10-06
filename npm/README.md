# npm

- [install](#install)
- [rm](#rm)
- [scripts](#scripts)
- [publish](#publish)

## install

```
npm install express
```

当然，也可以使用 `npm i express` 来安装，`i` 是 `install` 的简写

## rm

```
npm rm express --save
```
会卸载掉 `express` 模块，同时，会刷新 `package.json`

## scripts

```js
// index.js
const express = require('express');
const app = express();

const port = process.env.PORT || 3000;

app.get('/', (req, res, next) => {
  res.sned('hello');
});

app.listen(PORT, () => console.log(`app is running at localhost:${port}`));
```
可以在命令行里运行 `node ./index.js`

但还有别的方式，只需要在 `package.json` 的 `scripts` 里增加 `"start": "node ./index.js"`，然后，在命令行里运行 `npm run start` 即可


## publish

1 创建 `github`
```
git init

git config user.name sunsilently

git config user.email sunsilently@outlook.com

git remote add origin https://github.com/sileny/名称

git pull https://github.com/sileny/名称 main

git branch -m master main

git add .

git commit -m 注释

git push -u origin main
```

2 推送并关联github

2.1 创建 `package.json`
```
$ npm init
```
填写包信息

>比较重要的是：`name`、`version`

2.2 推送到 `npm` 仓库并关联到 `github`
```json
"repository": {
  "type": "git",
  "url": "git+https://github.com/sileny/docs.git"
},
```

2.3 发布
```
$ npm publish
```

或者
```
$ npm publish --access public
```
