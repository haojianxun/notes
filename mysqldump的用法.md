# mysqldump的用法

```
usage:
mysqldump [options] db_name [tbl_name ...]
mysqldump [options] --databases db_name ...
mysqldump [options] --all-databases

要是备份的是myisam引擎的话
-x --locck-all-tables 锁定所以库的所有表
-l --lock-tables 锁定制定库的所有表

要说innodb引擎的数据库的话
--single-transaction 创建一个事务 基于此快照执行备份


其他选项
-R --routines 存储过程和存储函数
--triggers
-E --events
--master-data[=#]
		1:记录CHANGER MASTER TO 语句,此语句不被注释
		2:记录CHANGER MASTER TO 语句,此语句会被注释
--flush-logs   锁定表之后,即可进行日志刷新
```

