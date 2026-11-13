# Git submodule 子模块
- 主项目并不是复制了一份子模块的代码，而是记录子模块的某个 commit SHA 信息
- 子模块不会自动跟着远端更新，如果要更新到新版本，需要主动操作

## 添加子模块
```bash
# 主项目 submodule-demo-app
cd submodule-demo-app
git submodule add https://github.com/louisgeek/submodule-demo-lib.git libs/submodule-demo-lib
git commit -m "Add submodule"
```

## 克隆主项目时一并初始化并拉取子模块（第一次拉项目方式1）
```bash
# 克隆 submodule-demo-app 主项目，并初始化、递归拉取子模块
# 子模块会检出到主项目当前 commit 所记录的特定 commit
git clone --recurse-submodules https://github.com/louisgeek/submodule-demo-app.git
```

## 之前已克隆了主项目，再初始化并拉取子模块（第一次拉项目方式2）
```bash
git clone https://github.com/louisgeek/submodule-demo-app.git

cd submodule-demo-app
# 将子模块检出到主项目当前 commit 所记录的子模块 commit
# --init：初始化尚未初始化的子模块
# --recursive：递归处理嵌套子模块
#
# 即使 .gitmodules 配置了 branch = main/develop，
# 此命令默认也不会追踪该分支的远端最新提交。
git submodule update --init --recursive
```

## 拉取主项目后，使子模块与主项目记录的版本对齐（日常拉代码）
```bash
# 拉取主项目
git pull
# 确保所有子模块（含嵌套子模块）都检出到主项目当前记录的 commit
git submodule update --init --recursive
```

如果已确认所有子模块都初始化过了，也可以通过一条命令实现
```bash
git pull --recurse-submodules
```

## 将子模块更新到其远端跟踪分支的最新提交
```bash
# 若 .gitmodules 配置 branch = develop，则跟踪 develop
# 若没有 branch 配置，则使用子模块远端 HEAD 所指向的默认分支（通常就是 origin/main）
git submodule update --remote --recursive
```

```bash
# 处理过 --remote 后，主项目中的子模块指针通常会发生变化
# 若要将该版本升级记录到主项目，需要提交一次主项目
git add path/to/submodule
git commit -m "update submodule"
git push
```