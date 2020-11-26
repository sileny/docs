# npm

- [install](#install)
- [rm](#rm)
- [scripts](#scripts)
- [publish](#publish)
- [version](#version)

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

git config user.name xxx

git config user.email yyy@mail.com

git remote add origin https://github.com/xxx/名称

git pull https://github.com/xxx/名称 main

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
  "url": "git+https://github.com/xxx.git"
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

## version

NPM 依赖包版本前面符号的意义

1 版本号的格式及意义
```
格式: X . Y . Z
大版本 . 次要版本 . 小版本
主版本号 . 次版本号 . 补丁版本号
````

2 对于版本号,再升级依赖包时该如何改动版本号呢？

```
大版本(主版本号(major)): 大变动,新的架构设计调整,向下不兼容,需要更新主版本号
次要版本(次版本号(minor)): 新增功能,向下兼容,需要更新次版本号
小版本(补丁版本号(patch)): 修复Bug,需要更新补丁版本号
```

3 版本号前面符号的意义


3.1 必须匹配某个版本

如: `2.2.1`, 表示必须依赖 `2.2.1` 版的依赖包

3.2 `>version`

必须大于某个版本

如: `>2.2.1`, 表示必须依赖大于 `>2.2.1` 版的依赖包

3.3 `>=version`

必须大于或等于某个版本

如: `>=2.2.1`, 表示必须依赖大于或等于 `>=2.2.1` 版的依赖包

3.4 `<version`

必须小于某个版本

如: `<2.2.1`, 表示必须依赖小于 `<2.2.1` 版的依赖包

3.5 `<=version`

必须小于或等于某个版本

如: `>=2.2.1`, 表示必须依赖小于或等于 `>=2.2.1` 版的依赖包

3.6 `~version`

不改变大版本号和次要版本号,小版本号随意

注意:
```
如果按照版本号格式,X.Y.Z,那么小版本号就是随意
如: ~2.2.1, 表示 >=2.2.1 <2.3.0 版的依赖包 (可以是2.2.1, 2.2.2, 2.2.3, …, 2.2.n)
如果版本号格式,X.Y,那么跟正规格式的意义相同
如果版本号格式,X,那么次要版本号和小版本号可以随意
如: ~2, 表示 >=2.0.0 < 3.0.0版的依赖包 (可以是2.0.0, 2.0.n, 2.1.0, …, 2.n.n)
```

3.7 `^version`

版本号最左边非 0 数字的右侧可以任意

如: `^2.2.1`,表示 `>=2.2.1 < 3.0.0` 版依赖包

`^0.2.1`,表示 `>=0.2.1 <0.3.0` 版依赖包

`^0.0`,表示 `>=0.0.0 <0.1.0`版依赖包

3.8 `version号位置出现 X`

X 的位置表示任意版本

如: `2.2.x`,表示 `>=2.2.0 <2.3.0` 版依赖包

3.9 version使用 * 代替

任意版本, *""*也表示任意版本

如: *, 表示 >=0.0.0版依赖包

3.10 `version(1) - version(2)`

大于等于version(1),小于等于version(2)

如: `2.2.1 - 2.3.1`, 表示 `>=2.2.1 <=2.3.1` 版依赖包

3.11、根据前面十条设置版本号规则, `range(1)||range(2)`

满足range(1)或者满足range(2),可以设置多个范围,用空格隔开

如: `<2.2.1||>=2.3.1 <2.4.1||>2.5.1 <2.6.1`,表示三个范围的版本号都可以
