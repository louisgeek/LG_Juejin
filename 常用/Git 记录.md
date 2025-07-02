


## 常用命令
```shell
Git 全局设置:
git config --global user.name "louisgeek"
git config --global user.email "louisgeek@qq.com"


创建 git 仓库:
mkdir test
cd test
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://git.oschina.net/louisgeek/test.git
git push -u origin main


已有项目?
cd existing_git_repo
git remote add origin https://git.oschina.net/louisgeek/test.git
git push -u origin main
```



```shell
 git add -A
fatal: detected dubious ownership in repository at 'D:/****/WWW/www.***.cc'
'D:/**/WWW/www.**.cc' is owned by:
        'S-1-5-32-544'
but the current user is:
        'S-1-5-21-4097290046-3821524887-*****-1001'
To add an exception for this directory, call:
 
        git config --global --add safe.directory D:/***/WWW/www.**.cc
```



Windows 目录权限不对