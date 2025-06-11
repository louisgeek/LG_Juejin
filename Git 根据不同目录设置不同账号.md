# Git 根据不同目录设置不同账号

## includeIf 配置
```shell
# 先创建 gitconfig 文件（如果不存在）
touch ~/.gitconfig
touch ~/.gitconfig_pers
touch ~/.gitconfig_work
```

```shell
# ~/.gitconfig

[core]
	longpaths = true # 允许处理长路径

# includeIf 的 gitdir 条件只会在实际 Git 仓库中生效，普通的文件目录不会触发条件包含
[includeIf "gitdir:C:/LouisPers/"] # 注意目录结尾不要少了 /，代表是个目录
    path = ~/.gitconfig_pers

[includeIf "gitdir:C:/LouisWork/"] # 注意目录结尾不要少了 /，代表是个目录
    path = ~/.gitconfig_work
```


```shell
# ~/.gitconfig_pers

[user]
    name = your_pers_name
    email = your_pers_email@example.com
```


```shell
# ~/.gitconfig_work

[user]
    name = your_work_name
    email = your_work_email@company.com
```