# command

- [init](#init)
- [push](#push)
- [更新代码后提交](#update)

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
git add . && git commit -m u && git push
```

