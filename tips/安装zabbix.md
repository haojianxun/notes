# 安装zabbix

## 使用阿里云镜像源

###替换默认centos源为阿里云的源

```
cd /etc/yum.repos.d/

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 使用阿里云的epel源

```
备份(如有配置其他epel源)

mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup

wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### 使用阿里云的zabbix镜像源

```
cd /etc/yum.repos.d

vim zabbix.repo

[zabbix]
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/3.4/rhel/7/x86_64/
enabled=1
gpgcheck=0
```

最后执行

```
yum makecache
```

## 安装zabbix

```
yum install -y zabbix-release    //安装源码库配置部署包

yum install -y zabbix-server-mysql zabbix-web-mysql



只安装Zabbix Agent的示例.
yum install zabbix-agent
```

## 安装mysql数据库

```
yum install -y mysql-server mysql

systemctl enable mysql
systemctl start mysql
```

##初始化mysql数据库

```
shell> mysql -uroot -p<password>
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
mysql> quit;
```

然后导入初始架构（Schema）和数据。

```
cd /usr/share/doc/zabbix-server-mysql-3.4*
zcat create.sql.gz | mysql -uroot zabbix
```

## 修改http配置

```
vim +95 /etc/httpd/conf/httpd.conf

ServerName 192.168.200.132:80

systemctl restart httpd 
```

## 修改zabbix时区

```
vim /etc/httpd/conf.d/zabbix.conf

#启用时区设置,并将其改为上海
php_value date.timezone Asia/Shanghai
```



## 启动Zabbix Server进程

在zabbix_server.conf中编辑数据库配置

```
# vi /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=
```

启动Zabbix Server进程

```
systemctl start zabbix-server
```

## 配置zabbix-agent

```
vim /etc/zabbix/zabbix_agentd.conf

#修改其中是Server项 , 让其指向zabbix-server的地址
Server=192.168.200.132
```



##访问zabbix

Zabbix前端可以在浏览器中通过 <http://zabbix-frontend-hostname/zabbix> 进行访问。默认的用户名／密码为 Admin/zabbix