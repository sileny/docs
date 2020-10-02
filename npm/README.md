# npm

- [publish](#publish)

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
