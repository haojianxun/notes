# httpd压力测试

curl命令

curl常用的选项

- -I /--head 只响应报文首部信息
- -A/--user-agent <string> 设置用户代理发送给服务器
- -basic 使用HTTP基本验证
- --tcp-nodealy 使用TCP_NODELAY选项
- --cacert <file> CA证书
- -H/--header <line>自定义投信息传递给服务器
- --limit-rate <rate> 设置传输速度
- -u/--user <user[:password]>设置服务器的用户和密码
- -0/--http1.0   使用HTTP/1.0

ulimit命令

- 软限制:可以超过的限制,但是仅仅可以超过一定时长
- 硬限制:绝对不能超过的限制
- 配置文件是/etc/security/limits.conf   扩展配置/etc/security/limits.d/*.conf
- -n [N] 显示或限定能打开的最大文件句柄数
- -u [N] 所能运行的最多进程数

ab命令  apache bench

- -c concurrency

  Number of multiple requests to perform at a time. Default is one request at a time.

  模拟的并发数  -c的值要小于-n的值

httpd压力测试环境

​	ab webbench http_load ,seige

​	jmeter loadrunner

​	tcpcopy 网易的  可以将生产环境中的真实请求保存下来

