# mysql管理员忘记密码

```
创建用户:
CREATE USER 'test'@'localhost' IDENTIFIED BY 'magedu';

修改用户密码:
SET PASSWORD FOR 'test'@'localhost'=PASSWORD('123456');

UPDATE mysql.user SET Password=PASSWORD('123456') WHERE User='test' AND Host='localhost';

mysqladmin -uUSERNAME -hHOST -p PASSWORD'NEW_PASS';


忘记管理员密码的方法:
在`/usr/lib/systemd/system/mariadb.service`   增加 --skip-grant-tables --skip-networking
之后systemctl daemon-reload   之后再systemctl restart mariadb.service 
进去mariadb之后
UPDATE mysql.user SET Password=PASSWORD('123456') WHERE User='test' AND Host='localhost';之后再把刚刚修改的去掉  再去systemctl deamon-reload   再systemctl restart mariadb.service
```

