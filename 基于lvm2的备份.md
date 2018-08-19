# 基于lvm2的备份

前提:数据位于逻辑卷,包涵数据文件和事务日志

```
请求锁定所有表
mysql> FLUSH TABLES WITH READ LOCK;

2.记录二进制文件事务位置
FLUSH LOGS;
SHOW MASTER STATUS;
mysql -e 'SHOW MASTER STATUS;' >> /PATH/TO/SOME_POS_FILE

3.创建快照卷
lvcrete -L # -s -p r -SNAM-NAME/dev/VG-NAME/LV-NAME

4.释放锁
UNLOCK TABLES

5.挂载快照卷 并执行备份 备份完成后删除快照卷
6.周期性备份二进制日志
```

