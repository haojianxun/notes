## ssh相关

- maxsessions:最大并发连接,而且是长连接



CentOS6

- /etc/rc.d/init.d/ssh

CentOS 7

- systemd unit file :/usr/lib/systemd/system/sshd.service



ssh最佳实践

- 不要使用默认端口
- 禁止使用protocol version 1
- 限制可登陆的用户
- 设定空闲会话超时时长
- 利用防火墙设置ssh访问策略
- 仅监听特定的ip地址
- 使用强密码
- 禁止使用空密码
- 禁止root用户直接登陆
- 限制ssh的访问频度和并发在线数
- 做好日志 经常分析

日志在/var/log/secure