# command

- [常用项目配置](basic.md)
- [permission](permission.md)
- [init](#init)
- [push](#push)
- [更新代码后提交](#update)
- [githooks](#githooks)

## init
```
git init && git config user.name sunsilently && git config user.email sunsilently@outlook.com && git remote add origin https://github.com/sileny/xxx && git pull https://github.com/sileny/xxx main && git branch -m master main
```

等同于顺序执行以下命令

```
git init

git config user.name sunsilently

git config user.email sunsilently@outlook.com

git remote add origin https://github.com/sileny/xxx

git pull https://github.com/sileny/xxx main

git branch -m master main
```

## push

```
git add . && git commit -m a && git push -u origin main
```

## update

```
git status && git add . && git commit -m u && git push -u origin main
```

## githooks

`husky` 是一个为 `git` 客户端增加 `hook` 的工具。安装后，它会自动在仓库中的 `.git/` 目录下增加相应的钩子，比如 `pre-commit` 钩子就会在你执行 `git commit` 的触发。

可以在 `pre-commit` 中实现一些比如 `lint` 检查、单元测试、代码美化等操作。当然，`pre-commit` 阶段执行的命令当然要保证其速度不要太慢，每次 `commit` 都等很久也不是什么好的体验。

`lint-staged`，一个仅仅过滤出 Git 代码暂存区文件(被 `git add` 的文件)的工具

Git Hooks 就是在 Git 执行特定事件（如 `commit`、`push`、`receive` 等）时触发运行的脚本，类似于“钩子函数”，没有设置可执行的钩子将被忽略。

在项目的 `.git/hooks` 目录中，有一些 `.sample` 结尾的钩子示例脚本，如果想启用对应的钩子，只需手动删除后缀，即可。删除某一个 `hook` 的后缀 `.sample` 即可启用该 `hook` 脚本，默认是不启用的

>但是，一般不去改动 `.git/hooks` 里面的文件，因为可以使用 `husky`

首先，需要使用 `git init` 创建 `.git` 目录。如果先安装 `husky`，后执行 `git init`，在 `.git` 目录不会存在 `husky` 创建的 `hook` 脚本

安装 `husky` 和 `lint-staged`
```
yarn add husky lint-staged -D
```

`husky` 在安装过程中会在 `.git/hooks` 文件夹中生成一系列的 git hook 脚本

`package.json` 添加如下配置

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
  ]
}
```
