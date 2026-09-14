# SMTP IMAP POP3
- Simple Mail Transfer Protocol 简单邮件传输协议，用于发送邮件
- Internet Message Access Protocol 互联网邮件访问协议，用于访问和同步邮箱，客户端可以直接操作服务器上的邮件（如删改、标记），所有修改在多设备间同步
- Post Office Protocol version 3 邮局协议第 3 版，也用于收取邮件，但理念和 IMAP 不一样（POP3 是单向下载，IMAP 是双向同步）



# StartTLS 和 SSL/TLS 区别
|  | StartTLS | SSL/TLS（TLS 直连） | 说明 |
|---|---|---|---|
| 别名 | 显式 TLS | 隐式 TLS | |
| 加密开始时机 | 先建立普通协议连接，再升级为 TLS | 建立 TCP 连接后立刻开始 TLS | |
| 连接过程 | 协议协商 → `StartTLS` 命令 → TLS 握手 | TLS 握手 → 协议通信 | |
| SMTP 常用端口 | 587（25 饱受封禁和限制） | 465 | 客户端向服务商提交待发送邮件 |
| IMAP 常用端口 | 143 | 993 | 收信、同步和管理邮箱 |
| POP3 常用端口 | 110 | 995 | 收信，将邮件下载到本地 |
| 客户端常见设置名 | `StartTLS` | `SSL/TLS`、`SSL`、`TLS` | |



## 最现代的组合
```
收信：IMAP 993，SSL/TLS
发信：SMTP 587，StartTLS
```

更推荐
```
收信：IMAP 993，SSL/TLS
发信：SMTP 465，SSL/TLS
```