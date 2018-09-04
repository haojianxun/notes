ftp

ftp file transfer protocol 文件传输协议 在应用层

基于c/s架构

服务端实现:vsftpd pureftpd proftpd  filezilla server

客户端实现:ftp,lftp ..(在Linux上) filezilla flashfxp(在windows上)



ftp的连接类型:

有2种连接方式

命令连接:建立会话的时候,了解服务器端和客户端的时候,客户端发起请求,服务器响应

数据连接:就是当你运行完成连接的时候  执行像什么cd啊 ls啊 等等命令的时候这个		时候就是数据连接,数据连接必然是通过某个命令连接发起的

主动(PORT style) ：服务器主动连接
命令（控制）：客户端：随机port --->服务器：tcp21
数据： 客户端：随机port+1  <--- 服务器：tcp20
被动(PASV style) ：客户端主动连接
命令 （控制 ）：客户端：随机port ---> 服务器：tcp21
数据：客户端：随机port+1 ---> 服务器：随机port

注意:主动与否是相对与服务器端来说的

服务器被动模式数据端口 示例：
227 Entering Passive Mode (192,168,175,138,224,59)
服务器数据端口 为：224*256+59



用户:资源位于用户的家目录下

匿名用户：ftp,anonymous, 对应Linux 用户ftp         (映射某一固定的系统用户)
系统用户：Linux 用户, 用户/etc/passwd, 密码/etc/shadow
虚拟用户：特定服务的专用用户，独立的用户/ 密码文件

响应码：
1XX ：信息 125 ：数据连接打开
2XX ：成功类状态 200 ：命令OK 230 ：登录成功
3XX ：补充类 331 ：用户名OK
4XX ：客户端错误 425 ：不能打开数据连接
5XX ：服务器错误 530





