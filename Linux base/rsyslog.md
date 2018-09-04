rsyslog

配置文件

/etc/rsyslog.conf    /etc/rsyslog.d/*.conf

主程序:/usr/sbin/rsyslogd

配置文件格式

- MOUDLES ;模块配置
- GLOBAL DIREECTIVES 全局配置
- RULES 日志记录相关的配置

RULES:

配置格式

faclilty.priority   target

facility

- \*  所有级别
- f1,f2..  指定的facility

priority:

- \* 所有级别
- none 没有级别
- PRIORITY 指定级别以上的所有级别
- =PRIORITY 仅记录指定级别

target

- 文件 将日志信息记录到指定的文件中 文件路径前的-表示异步写入
-  用户 将日志事件通知给指定用户
- 日志服务器 @host  把日志通知网络送到指定的服务器记录 而非由本地记录
- 管道 |

配置rsyslog成为日志服务器

\### MODULES\###

\# Privides udp syslog reception

$Modload imudp

$UDPServerRun 514

\### MODULES\###

\# Privides TCP syslog reception

$Modload imtcp

$InPutTCPServerRun 514

其他日志文件

/var/log/secure 系统安装日志 应该周期分析

/var/log/btmp   当前系统上 用户失败尝试登陆相关的日志信息 lastb命令查看

/var/log/wmp  正常登陆信息 last命令查看

lastlog命令 查看每一个用户最近一次登陆信息

/var/log/messages 系统日志信息

/var/log/dmesg 系统引导过程的日志信息

rsyslog将日志记录到mysql中

1. 装备MySQL Server

2. 授权 

   ```
   mysql > GRANT ALL ON Syslog.* TO 'USER'@'HOST' IDENTIFIED BY 'PASSWD'
   ```

3. yum -y install rsyslog-mysql

4. 导入rsyslog-mysql中的sql脚本  用rpm -ql rsyslog-mysql

5. 配置rsyslog

   \####MODULES\###

   $ ModLoad ommysql

   \####RULES\###

   facility.priority :ommysql:DBHOST,DBNAME,DBUSER,DBUSERPASSWD

6. 重启rsyslog服务

通过loganalyzer展示数据库日志

1. 准备amp或者nmp组合

2. yum install httpd php php-mysql php-gd

3. 安装LogAnalyzer

   tar xf 

   cp -a 