tcp_wrapper

判断一个服务程序是否能够由tcp_wrapper进程访问控制的方法

- 动态链接到libwrap.so库

  ldd /PATH/TO/PROGRAM

- 静态编译libwrap.so库文件到程序中:

  stringss /PATH/TO/PROGRAM

配置文件 /etc/hosts.allow /etc/hosts.deny

配置文件语法

```bash
daemon_list : client_list : option : option ...
```

deamon_list 程序文件名称列表

- 单个应用程序文件名
- 程序文件名列表,以逗号分隔
- ALL 所以受tcp_wrapper控制的应用程序文件

client_list

- 单个ip地址或主机名

- 网络地址

- 内建ACL

  ALL

  LOCAL

  UNKONWN

  PARANOID

  - EXCEPT

- [:option:option...]

  deny 拒绝 主要用于hosts.allow中拒绝规则

  allow 运行 主要用于hosts.deny中定义的允许规则

  spawn:生成之意 触发执行用户指定的任意命令 此处通常用于记录日志

  ```
  vsftpd:172.16.:spawn /bin/echo $(date)login attempt from %c to %s
  ```

  ​

sshd 172.16.0.0/255.255.0.0 EXCEPT 172.16.0.0/255/255.255.0 EXCEPT 172.16.0.200:spawn /bin/echo