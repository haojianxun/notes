# mysql主从复制配置

主服务器:

```
#/etc/my.cnf.d/server.cnf

server_id=1
log_bin=bin-log

启动服务:
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'repluser'@'192.168.*.*' IDENTIFIED BY '123456';

FLUSH PRIVILEGES;
```

从服务器

```
server_id=2
relay_log=relay-log

启动服务
CHANGE MASTER TO MASTER_HOST='HOST',MASTER_USER='USER',MASTER_PASSWORD='PASSWORD',MASTER_LOG_FILE='BINLOG',MASTER_LOG_POS=#;

START SLAVE[IO_THREAD|SQL_THREAD]

SHOW SLAVE STATUS;
```





##可能出现问题

`GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'repluser'@'192.168.*.*' IDENTIFIED BY '123456';`

之后会出现 *The MariaDB server is running with the --skip-grant-tables option so it cannot execute this statement*

解救方法:`flush privileges;`

