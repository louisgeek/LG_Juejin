# Git 查询 Tag 列表.md
- 很奇怪，TortoiseGit 和 SourceTree 怎么都没有直观展示区分远程和本地 Tag 的功能

## 查询远程 Tag
- 过滤关键字不区分大小写
```bash
# Git Bash
git ls-remote --tags origin | grep -i "release_"
# Cmd
git ls-remote --tags origin | findstr /i "release_"
# PowerShell
git ls-remote --tags origin | Select-String "release_"
```

- 区分大小写
```bash
# Git Bash
git ls-remote --tags origin | grep "release_"
# Cmd
git ls-remote --tags origin | findstr "release_"
# PowerShell
git ls-remote --tags origin | Select-String -CaseSensitive "release_"
```

## 查询本地 Tag
- 过滤关键字不区分大小写
```bash
# Git Bash
git show-ref --tags  |  grep -i "release_"
# Cmd
git show-ref --tags  |  findstr /i "release_"
# PowerShell
git show-ref --tags  |  Select-String "release_"
```

- 区分大小写
```bash
# Git Bash
git show-ref --tags  |  grep "release_"
# Cmd
git show-ref --tags  |  findstr "release_"
# PowerShell
git show-ref --tags  |  Select-String -CaseSensitive "release_"
```