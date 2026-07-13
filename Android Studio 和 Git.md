# Android Studio 和 Git

【工作区】→ 【暂存区】→【本地版本库】→ 【远程仓库】


## AS 常用图形化操作

Commit 窗口勾选各个文件 → 点击 Commit **提交**（其实是执行了 add 和 commit 两个命令，其弱化了【暂存区】的概念）

Git 列表里选中最后一次提交记录 → 右键点击 Undo Commit **撤销提交**（其实是执行了 reset --soft \<倒数第 2 次的提交 id>）


## 在 dev 分支代码写了一半（不想 commit），临时需要切 main 分支去改 bug
- 可以利用 Shelf 或者 Git Stash 实现

```bash
# 临时存档，把当前写了一半的代码临时保存起来，让【工作区】和【暂存区】立马变干净，方便切分支
git stash

git switch main

# 修复 main 分支上的 bug，并进行正常的提交流程（add & commit），修复完切回
git switch dev

# bug 改完后切回来，把临时存档的代码取回释放出来，继续接着写
git stash apply # 恢复临时存档的代码，不删除 stash 记录
# git stash pop # 在 apply 的基础上，删除 stash 记录，发生冲突时也会删掉
```

​

## 恢复还没 add 的，代码写了一半想直接丢弃所有变动（已跟踪文件的删、改）

```bash
# 用本地历史（HEAD）的代码，强行覆盖【工作区】正在写的代码
# 删除的文件（从已删除到恢复原文件）、修改的文件（从修改到恢复原始内容）、新增的文件保持原样
git restore .
# 旧版命令（-- 是参数分隔符，用于明确区分文件路径参数和分支名称参数，避免歧义）
# git checkout -- .

# 比如针对所有 .g.dart 文件，放弃修改（** 递归匹配零个或多个目录层级，* 匹配当前目录级别内的任意字符）
git restore "**/*.g.dart"
# 旧版命令
# git checkout -- "**/*.g.dart"
```

​

## 恢复已经 add 的（如果是 AS 操作，这个情况涉及的就比较少了）

```bash
# 把当前目录下所有变动（包括增删改）都放进【暂存区】
git add .

# 从【暂存区】移回【工作区】（恢复 git add . 产生的行为）
git restore --staged .
# 旧版命令
# git reset HEAD .
```

​

## 重置已经 commit 的（AS 进行 Undo Commit 撤销提交，撤销最近一次的提交操作，让代码回到提交前的状态）

```bash
# 提交，在【本地版本库】生成历史
git commit -m "备注"

# 撤销最后一次 commit（从当前最新的 HEAD 往前退 1 步）
git reset --soft HEAD~1
# 撤销最后一次 commit（精确处理，目标 id）
git reset --soft <倒数第 2 次的提交 id>

# 附：如果只是单纯的想改提交备注
git commit --amend -m "新的备注"
```


## 放弃未提交修改，回到最新提交的状态（【工作区】和【暂存区】直接丢弃所有变动）


```bash
# 彻底放弃所有本地修改，强制恢复到最后一次提交的干净状态（当前最新提交）
git reset --hard HEAD
# HEAD 和 HEAD~0 等价
#git reset --hard HEAD~0
```


## 删除最新提交，回到上一次提交（AS 进行 Drop Commit 删除提交，丢弃最近一次的提交操作，代码改动直接消失）
- 如果已经 push 过，需要 git push --force，多人协作谨慎使用
```bash
# 丢弃本次提交的改动
git reset --hard HEAD~1
```



## 整理多次提交历史，合并成一次提交，使日志更整洁清晰（AS 用 Squash Commits 将多个提交合并压缩为一个提交）

```bash
# 从指定的提交 id 之后开始（不包含这个提交 id）
git rebase -i <commit hash>
```

交互式变基

```bash
# log 日志，新的在上，旧的在下
# abc1111  fix bug  3       ← 最新的提交
# def2222  fix bug  2
# ghi3333  fix bug  1       ← 想把这3个合并
# 62f437fe feat: add login  ← 基点（保留）

# 合并3 个提交，需要指定目标提交的前一个提交 id（基点）
git rebase -i 62f437fe

# 或者合并最近 3 个提交
# git rebase -i HEAD~3
```

编辑器交互（AS 自动生成，只需要在弹出框里编辑新的 commit message 即可）

```bash
# 编辑器交互打开后展示原来的内容（注意编辑器内容的顺序是反过来的）
pick ghi3333 fix bug 1
pick def2222 fix bug 2
pick abc1111 fix bug 3   # 最新的提交

# 修改为以下内容
pick ghi3333 fix bug 1   # 完整保留该提交
squash def2222 fix bug 2 # 合并到上一个提交，保留 commit message（可重新编辑），如果改用 fixup 则静默丢弃message
squash abc1111 fix bug 3 # 合并到上一个提交，保留 commit message（可重新编辑），如果改用 fixup 则静默丢弃message


# 保存退出 (:wq)
```

​
# Shelf 和 Git Stash
- Shelf 支持部分文件、部分代码块存储，不依赖 Git，Shelve Silently 临时存档 Unshelve Silently 取回临时存档
- Shelve Changes...，就是 Git Stash，必须整个工作区一起存，但支持协作，换电脑也能继续下去