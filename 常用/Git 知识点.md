# Git 知识点
- Workspace：工作区，用来编辑保存项目文件的地方
- Index/Stage：索引（暂存区），用于临时存放你的改动，保存了下次将提交的文件列表信息，通常存放在 /.git/index 文件中
- Repository：仓库区（本地版本库），存放在 /.git 目录中，包含项目的所有版本历史记录（每次提交的快照），其中 HEAD 指向最近一次提交的快照
- Remote：远程仓库，存放在远程服务器

## 全局配置文件路径
```shell
# git config --global 保存位置
C:\Users\<User>\.gitconfig
```

## config
```shell
git config  user.name "louisgeek"
git config  user.email "louisgeek@qq.com"
# 全局
git config --global user.name "louisgeek"
git config --global user.email "louisgeek@qq.com"
# sslVerify
git config --global http.sslVerify "false"

# 查询配置
git config --list
# 查询全局配置
git config --global --list

# 取消配置
git config --global --unset http.sslbackend
```

## 常用命令
```shell
# 在当前目录下创建 git 库
git init
# 添加指定文件
git add README.md
# 添加所有文件
git add .
# 提交文件
git commit -m "修改文件" # 描述
# 推送
git push -u origin main
```

## log
```shell
git log
# 简洁
git log --oneline
# 长 hash
git log --pretty=oneline
```

## commit
- -- amend 修改最近一次提交的描述信息、作者信息等
```shell
# 修改最近一次提交的描述信息
git commit --amend -m "修改最近一次提交的描述"
# 修改最近一次提交的作者信息，会进入编辑模式（i 进入 insert 模式，esc 退出 insert 模式。:wq 写入退出）。可以修改提交描述
git commit --amend --author "louisgeek <louisgeek@qq.com>" # 不带空格也行，git 会自动加空格
# 不进入编辑模式
git commit --amend --author "louisgeek <louisgeek@qq.com>" --no-edit
# 同时修改
git commit --amend -m "修改最近一次提交的描述" --author="louisgeek <louisgeek@qq.com>"
```

## push
- -u 参数设置本地分支的 upstream branch 上游分支，上游分支是指本地分支与远程分支之间的关联关系，设置上游分支后，Git 会记住这个关联关系，方便后续的操作
```shell
git push <远程主机名> <本地分支名>:<远程分支名>
git push origin main:main
# 如果本地分支名与远程分支名相同，则可以省略冒号
git push <远程主机名> <本地分支名>
git push origin main

# 推送并设置上游分支
git push -u origin main
# 后续可以直接推送
git push
#
```

## checkout
- 用于切换分支/恢复文件
- 用 HEAD 表示当前版本（也就是最新的提交），上个版本就是 HEAD^（也可以写成 HEAD~1），上上个版本就是 HEAD^^，当然往上 100 个版本写 100 个 ^ 容易出错，所以可以写成 HEAD~100
```shell
# 切换分支
git checkout develop # 切换到 develop 分支
git checkout -b new-branch # 创建一个新的分支并立即切换过去

# 恢复文件
# 将文件恢复到最新的提交的状态
git checkout HEAD -- README.md  # 相当于丢弃未提交的修改（改乱了，不要了，等同于 AS Git 里的 Rollback 功能）
# 将文件从 develop 分支覆盖到工作区
git checkout develop -- README.md
# 将文件从指定提交覆盖到工作区
git checkout <commit-hash> -- README.md

# 恢复工作目录中的文件
git checkout README.md # 丢弃对 README.md 的未提交修改

# 撤销暂存区的修改
git checkout -- README.md # 丢弃对 README.md 的未提交修改

# 切换到某个提交，将 HEAD 指向某个具体的提交（而非分支），进入分离头指针状态，提交不关联任何分支
git checkout <commit-hash> 

# 创建孤立分支​（适用于重构项目）
git checkout --orphan dev-branch # 创建无历史记录的新分支

# 专门用于切换分支
git switch develop # 切换分支
git switch -c new-branch  # 创建并切换分支

# 专门用于恢复文件
git restore README.md     # 恢复文件

# 取消暂存（即从暂存区移除）。该命令将已经通过’git add’命令添加到暂存区的文件恢复到工作区中。
git restore --staged README.md
```

## reset
- 重置当前 HEAD 到指定的状态
```shell
# --soft 仅移动 HEAD 指针到指定提交，保留暂存区（索引）和工作区的更改
git reset --soft HEAD~1 # 撤销提交，但保留修改在暂存区
git commit -m "新提交消息"  # 可直接提交，相当于修改最近一次的提交信息

# --mixed 移动 HEAD 指针到指定提交，并重置暂存区（索引），但保留工作区的更改
git reset --mixed HEAD~1 # 撤销提交并将修改放回工作区，--mixed 默认可以省略
git add . # 需要重新暂存部分文件
git commit -m "更新文件"  # 重新提交

# --hard 移动 HEAD 指针到指定提交，并重置暂存区（索引）和工作区，丢弃未提交的更改（需谨慎使用）
git reset --hard HEAD~1  # 撤销提交并清空所有未提交的修改
```

## 修改 .gitignore 文件后立即生效
```shell
# 清除缓存
git rm -r --cached .
# 重新 trace file
git add .
# 提交
git commit -m "修改 .gitignore 文件"
# 可选，按需同步到 remote
git push origin main
```

## 保存未提交的修改
```shell
# 暂存修改
git stash          
git checkout branch
git stash pop      # 恢复暂存
```

## 删除仓库所有提交的历史记录，成为一个干净的新仓库

```shell
# 1.Checkout
git checkout --orphan latest_branch
# 2. Add all the files
git add -A
# 3. Commit the changes
git commit -am "commit message"
# 4. Delete the branch
git branch -D main
# 5.Rename the current branch to main
git branch -m main
# 6.Finally, force update your repository
git push -f origin main
```











