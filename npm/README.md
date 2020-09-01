# npm

- [publish](#publish)

## publish

1 创建 `github`
```
git init

git config user.name username
git config user.email emailAddress

git remote add origin https://github.com/sileny/docs

git add .

git commit -m 注释

git push -u origin master
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
