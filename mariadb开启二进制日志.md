## mariadb开启二进制日志

```
编辑mariadb服务器文件

vim /etc/my.cnf.d/server.cnf

在[mysqld]下面加上
log_bin='LOGNAME'   //LOGNAME是你要填的二进制文件的文件名

之前重启mairadb(开启日志功能要在启动之前)
systemctl restart mariadb
```

