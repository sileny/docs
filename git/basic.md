# 基本操作
https://www.yiibai.com/git/git_environment.html

1、初始化
```sh
git init
```

2、设置本地配置
```sh
git config user.username name
git config user.email email
```

3、拉取代码

3.1、没有更改过代码
```sh
# 第一次拉取
git pull http://192.168.30.250/landsky/front-team.git
# 非第一次
git pull
```

3.2、更改过代码

```sh
# 提交前先把本地修改存入栈；
git stash

# 然后从服务器端更新代码；
git pull

# 还原暂存的内容
git stash pop stash@{0}

# 有冲突解决冲突

# 将暂存的内容恢复并在堆栈中删除它(这个指令将缓存堆栈中的第一个stash删除，并将对应修改应用到当前的工作目录下)。
git stash pop

# 查看stash
git stash list

# 移除stash
git stash drop stash@{0}

# 清除所有的stash
git stash clear
```

4、提交

```sh
查看文件或文件夹状态
git status
执行上面即可，没有操作示例

添加需要提交的文件到缓冲区
git add filename

添加需要提交的文件夹到缓冲区
git add dirname/
或者
git add dirname/*

再次查看文件或文件夹的状态
git status

如还有红色的，可以再次执行add命令

git commit -m"提交的内容描述"

如果是第一次提交到git服务器(只使用一次)
git remote add origin https://github.com/sileny

推送数据到git服务器的master仓库(第一次推送)
$ git push -u origin main
```

5、删除文件
```sh
git rm file
git commit -m"delete file"
git push
```

6、撤销操作

6.1、-amend

在任何一个阶段，都可能要撤销某些操作，可以加上 --amend

```sh
git commit -m 'initial commit'
git add forgotten_file
git commit --amend
```

最终只有只会提交第二次的结果。


6.2、取消暂存的文件

如果已经修改了两个文件并且想要将它们作为两次独立的修改提交，
但却意外地输入了 `git add *`暂存了两个。可以取消其中的一个。

```sh
git add *
git status
git reset HEAD filename
git status
```

6.3、撤销对文件的修改

还原代码为克隆时的状态。

```sh
git status
git checkout -- filename
ls
```

6.4、撤销更改
```sh
git log

# 会从磁盘删除(代码不会保留)
git reset --hard COMMIT_ID

# 代码仍然保留在本地
git reset --soft COMMIT_ID

# 不执行以下命令可能会有以下错误提示。
error: Your local changes to the following files would be overwritten by merge
git stash
```

6.5、撤销merge后的更改
```sh
git revert -m commitId
```

6.6 撤销所有更改
```sh
git clean -df
git reset --hard
```

# 删除文件夹

```sh
git rm -r -f --cached .idea
git commit -m comment
git push -u origin master
```


# 提交到别的分支

```sh
git cherry-pick commitId
git push
```


# 删除配置项

```sh
git config --local --unset user.username
git config --local --unset user.email
```

# 移动文件
```sh
git mv filename dirname/
```

# 重命名
```sh
git mv filename1 filename2
```

# 分支

查看分支
```
git branch
```

创建分支
```
git branch branchName
```

合并分支
```
git merge origin/branchName
```

切换分支
```
git checkout -b branchName
```

`-b` 此操作将创建一个新的分支，并立即切换到新分支

删除分支
```
// 删除本地
git branch -D branchName

// 删除远程
git push origin --delete branchName
```

重命名分支
```
git branch -m oldName newName
```

新仓库拉取指定分支
```
git init
git branch
git checkout -b branchName
git pull
git branch
```

# log
```
git log --author=admin
git shortlog
```

# 查看指令如何使用

```sh
git help config
```

# git文档地址
https://git-scm.com/docs
