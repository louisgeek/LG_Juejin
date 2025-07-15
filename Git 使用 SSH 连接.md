
# Git 使用 SSH 连接

## ssh-keygen
- SSH key generate，用于生成 SSH 密钥对文件（包含公钥和私钥），用于安全认证（比如远程登录、Git 操作等）
```shell
# -f <filename>  设置密钥保存的文件名（包括路径）
# -t <type> 设置密钥算法类型（比如 rsa）
# -b <bits>  设置密钥长度（比如 4096）
# -C <Comment>  设置注释信息（通常为邮箱或用途描述）
ssh-keygen -t <密钥算法类型> -b <密钥长度> -C "<注释>"

# 密钥文件默认生成在用户的 $Home 目录下的 .ssh 文件夹中
# C:\Users\<User>\.ssh\id_<算法>
# C:\Users\<User>\.ssh\id_<算法>.pub

# 自定义路径（~ 代表用户的 $Home 目录）
ssh-keygen -f ~/your_ssh_dir/your_key_name

# 全默认
ssh-keygen

# 指定 rsa 类型
ssh-keygen -t rsa
```

## ssh-agent
- SSH 密钥管理工具（代理），主要是用于缓存 SSH 私钥的密码，简化身份验证流程
```shell
# 整体就是启动 ssh-agent 代理，并将环境变量（SSH_AUTH_SOCK 和 SSH_AGENT_PID）注入当前 Shell 会话，使得当前 Shell 可以使用 SSH 代理管理密钥
# -s 表示让 ssh-agent 输出与 sh 兼容的 Shell 脚本语法（即生成用于设置环境变量的脚本）
# $(ssh-agent -s) 会先执行 ssh-agent -s，并将其输出结果捕获为一个字符串
# eval 命令会将捕获的字符串作为 Shell 命令执行
eval "$(ssh-agent -s)"
```

## ssh-add
- 将生成的 SSH 密钥文件添加到 ssh-agent，交给 ssh-agent 代理保管（将认证申请流程交由 ssh-agent 来完成），通过 ssh-add 添加私钥后，ssh-agent 会缓存密钥，后续 SSH 操作（比如 git clone）可自动认证，避免每次输密码
```shell
# 将私钥加载到代理（若私钥带密码短语，需输入一次密码，输入一次后缓存）
ssh-add ~/.ssh/id_rsa

# 查看已加载的私钥
ssh-add -l

# 移除指定密钥
ssh-add -d ~/.ssh/id_rsa

# 清空所有密钥
ssh-add -D
```

## ssh config 配置
```shell
# touch 创建 config 文件
touch ~/.ssh/config
```

```shell
# ~/.ssh/config

Host 192.168.x.x
    HostName 192.168.x.x # 主机地址
    IdentityFile ~/.ssh/id_rsa # 认证文件路径
```

## 步骤
```shell
# 1 创建密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# 2 启动 ssh-agent 并加载环境变量
eval "$(ssh-agent -s)"
# 3 添加密钥到 ssh-agent
ssh-add ~/.ssh/id_rsa
```

```shell
# 4 编辑 config 文件内容

# ~/.ssh/config

Host gitwork # alias
    HostName 192.168.x.x
    IdentityFile ~/.ssh/id_rsa
```

## 多账号
```shell
# 创建个人账号的密钥
ssh-keygen -t rsa -b 4096 -C "your_pers_email@example.com" -f ~/.ssh/id_rsa_pers
# 创建工作账号的密钥
ssh-keygen -t rsa -b 4096 -C "your_work_email@company.com" -f ~/.ssh/id_rsa_work
```

```shell
# ~/.ssh/config

Host gitpers # alias
    HostName github.com
    User your_pers_name
    IdentityFile ~/.ssh/id_rsa_pers

Host gitwork
    HostName 192.168.x.x
    User your_work_name
    IdentityFile ~/.ssh/id_rsa_work
```


