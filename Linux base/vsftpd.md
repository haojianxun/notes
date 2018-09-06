vsftpd

local_enable  #所有的非匿名用户

local_umask=022  本地用户上传文件的权限掩码

目录消息

dirmessage_enable=YES

> 用户第一次进入目录时,vsftpd会扫描目录下的.message文件,并展现给用户
>
> message_file 指定文件路径,而不使用默认的.message

数据传输日志

xferlog_enable

xferlog_str_format

xferlog_file=/var/log/xferlog

数据传输模式

connect_from_port_20 是否启用port模式

修改匿名用户上传的文件的属主

chown_uploads 是否修改

chown_username 启用chown_uploads指令时,将文件属主修改为指定的用户 ,默认是root

chown_upload_mode 设定匿名用户上传的文件权限 默认为600

设定会话超时时长

idle_session_timeout 空闲会话超时时长

connect_timeout:PORT下 服务器连接是超时时长

data_connection_timeout

命令连接的监听端口

listen_port 默认是21

设定连接和传输速率

local_max_rate 本地用户的最大传输速率 单位是字节 默认是0,表示无限制

anon_max_rate 匿名用户的最大传输速率

max_clients 最大并发连接数

max_per_ip 每个ip所允许发起的最大连接数

禁锢本地用户

chroot_local_user=YES  禁锢所有本地用户 只能在自己家目录下,不能对自己家目录有写权限

chroot_list_enable=YES  禁锢指定用户在家目录

chroot_list_file



userlist_enable=YES 启用时,vsftpd将加载一个由userlist_file指令指定的用户列表文件,此文件中的用户能否访问vsftpd服务取决于userlist_deny指令,userlist_deny=YES,表示此列表为黑名单,等于no则表示一个白名单,userlist_deny默认是yes