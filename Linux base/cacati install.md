# cacati install

```
一、准备工作
环境：Centos 5.6
所需软件：
http
Php
Php-mysql
Php-snmp
Mysql
Perl-DBD-MySQL
Php-pdo
rrdtool
Net-snmp
Net-snmp-libs
Net-snmp-utils
#下载相关软件
cd /usr/local/src/
wget http://www.cacti.net/downloads/cacti-0.8.7e.tar.gz
二、环境介绍
主监控机是Centos 5.6
主监控机IP=192.168.0.61
三、安装配置
（1）在主监控机上安装apache+php+gd的web环境,推荐编译安装，不再赘述，本处方便起见用yum装了


yum install php php-mysql php-snmp mysql mysql-server net-snmp net-snmp-libs net-snmp-utils php-pdo perl-DBD-MySQL

（2）在主监控机上安装rrdtool,rrdtool依赖的包过多，所以选择增加源，然后用yum安装
安装rrdtool  rrdtool rrdtool-devel rrdtool-php

服务器下载下来然后使用一下命令
yum localinstall --nogpgcheck

（3）配置snmp
vi /etc/snmp/snmp.conf
#将下边这行中的default
com2secnotConfigUser default public
#改为127.0.0.1
com2secnotConfigUser 127.0.0.1 public
#将下边这行中的systemview
access notConfigGroup "" any noauth exact systemview none none
#改为all
access notConfigGroup "" any noauth exact all none none
#将下边这行的注释“#”号去掉
#view all included .1 80

service snmpd restart

（4）安装cacti
#把解压后的包移动到你的相应的web目录
tar xvf cacti-0.8.7e.tar.gz
mv cacti-0.8.7e /var/www/html/cacti
（5）在数据库中建库、授权、导入数据库结构
#注意导入cacti.sql时该文件的路径
mysql -p
mysql> create database cacti;
mysql> grant all privileges on cacti.* to cacti@localhost identified by 'cacti' with grant option;
mysql> grant all privileges on cacti.* to cacti@127.0.0.1 identified by 'cacti' with grant option;
mysql> use cacti;
mysql> source /var/www/html/cacti/cacti.sql;
#配置cacti以连接数据库
vi /var/www/html/cacti/include/config.php
（6）浏览器下配置
#用浏览器打开 http://192.168.0.61/cacti ，会显示 cacti的安装指南，设置好就不会再出现了。
#点击 “Next”
#选择“New Install”，点击“Next”
#指定 rrdtool、 php、 snmp 工具的 Binary 文件路径，确保所有的路径都是显示“ FOUND”，没有 “NOT FOUND”的，点击 Finish 完成安装。
#Cacti 默认的用户名与密码是 admin，输入用户名与密码，点击 login
#为了安全的原因，第一次登录成功后，cacti 会强制要求你更改一个新的 password ，输入新密码并确认密码，点击 save ,进入 cacti 控制台界面：
#点击 graphs ，查看cacti 监控本机的图表：
（7）增加入一个计划任务，使得 cacti 每五分钟生成一个监控图表。
crontab -e
#加入如下内容。注意poller.php的路径
*/5 * * * * php /var/www/html/cacti/poller.php > /dev/null 2>&1
#确保 /var/www/html/cacti/rra/目录存在
#如果暂时未看到图表，可以手工执行，生成图表
#php /var/www/html/cacti/poller.php > /dev/null 2>&1
（8）使用 Cacti 监控 Linux 主机
#在被监控的linux主机上安装net-snmp
yum install net-snmp
vi /etc/snmp/snmpd.conf
#更改以下部分
#将下边这行中的default
com2secnotConfigUser default public
#改为10.0.0.52（cacti）服务器的地址)
com2secnotConfigUser 10.0.0.52 public
#将下边这行中的systemview
access notConfigGroup "" any noauth exact systemview none none
#改为all
access notConfigGroup "" any noauth exact all none none
#将下边这行的注释“#”号去掉
#view all included .1 80
service snmpd restart
（9）如果出现问题请注意一下snmp协议的版本，都用version 1是一种解决方法
如果都用version 1,需要把所有监控机和被监控机的snmpd.conf改一下
#vi /etc/snmp/snmpd.conf
#将下边这行
view systemview included .1.3.6.1.2.1.1
#改为
view systemview included .1.3.6.1.2.1


二、cacti常用插件安装
要安装别的插件前,先要安装cacti的一个patch－－Plugin Architecture,才能支持插件
PA 2.8 = cacti 0.8.7g

# tar xvf cacti-plugin-0.8.7g-PA-v2.8.tar.gz
# cp -R cacti-plugin-arch/* /var/www/html/cacti/
cd /var/www/html/cacti/
mysql -ucacti -pcacti cacti <pa.sql
patch -p1 -N <cacti-plugin-0.8.7g-PA-v2.8.diff
vi include/config.php
修改为$url_path = “/cacti/”;
登陆，启用PA。

安装常用插件
Monitor,Settings,thold

# tar zxvf monitor-latest.tgz
# tar zxvf settings-latest.tgz
# tar zxvf thold-latest.tgz
# mv monitor-0.9/ /var/www/cacti/plugins/monitor
# mv settings-0.6/ /var/www/cacti/plugins/settings
# mv thold-0.41/ /var/www/cacti/plugins/thold
登陆安装启用对应的插件即可。


六、常见故障排除
安装完毕在浏览器上无法看到数据的png图片。看apache的log 
    如果出现：
    ========================
    [Thu Feb 09 15:12:24 2006] [error] [client 127.0.0.1] File does not exist: /var/www/html/favicon.ico
    ERROR: opening '/var/www/html/cacti/rra/localhost_mem_buffers_3.rrd': Permission denied
    PS:解决办法：关闭selinux，即可解决问题。

PS：以上无法获取数据图大多和poller.php，cmd.php权限有关。
当cacti 有图没有数据时，而且状态为nan的错误
PS：这个很可能是snmp的问题,执行以下命令，没有得到如图的结果。就说明snmp不支持64位MIB库。重新编译安装snmp
# snmpwalk -c public -v 2c 127.0.0.1 IF-MIB::ifHCInOctets
IF-MIB::ifHCInOctets.1 = Counter64: 7437357
IF-MIB::ifHCInOctets.2 = Counter64: 353773IF-MIB::ifHCInOctets.3 = Counter64: 0
PS：被监控主机无法获得snmp信息，还有可能是对方主机snmp版本和当前主机的snmp版本不一致导致的。
PS：rrdtools版本要一致，特别是在升级cacti时候。版本不一致，可能rra数据格式不同。就无法处理。
排错思路
1，查看log下的日志文件。一般那里会有提示
2，测试SNMP是不是工作正常 snmpwalk -v 2c -c public hostIP   if正常的话会出现一些数据。不正常会出现一些错误，也会有对应的错误提示。
3，自动运行poller.php没有。有没有加入cacti的的用户。。有没有给cacti用户写入rra/ log/的权限。。
4，crontab –u cactiuser –e 为cactiuser加上自动运行poller.php的任务：*/5     *       *       *       *       root    /usr/local/bin/php /usr/local/share/cacti/poller.php /dev/null 2>&1
	5分钟刷新一次数据。你也可以根据需要还设置。
5。把cacti目录里的cmd.php和poller.php文件加下运行的权限。

=
脚本下载以及设置

wget http://mysql-cacti-templates.googlecode.com/files/mysql-cacti-templates-1.1.2.tar.gz
tar -xzvf mysql-cacti-templates-1.1.2.tar.gz
cd mysql-cacti-templates-1.1.2
cp ss_get_mysql_stats.php /xok.la/cacti/scripts
可以看到里面有多个监控项目，报告监控apache和nginx.我这只测试mysql,mysql相关的就2个文件：
模板文件：cacti_host_template_x_db_server_ht_0.8.6i.xml
插件：ss_get_mysql_stats.php

修改ss_get_mysql_stats.php 文件 第30行

$mysql_user = 'cacti';
$mysql_pass = 'cacti';
$cache_dir  = "var/www/html/cacti/cache/";
设置准备监控的数据库的账户相关信息

mkdir /xok.la/cacti/cache/

chmod 777 -R /xok.la/cacti/cache/
默认在获取的数据/tmp/下，会有cacti不能读取的情况。所以放在cacti目录来。

二，创建监控Mysql需要的账户以及权限
配置MySQL服务器，让cacti所在机器能够访问MySQL服务器的状态信息，必须拥有”process”权限。如果要监控InnoDB状态，还必须有”SUPER”权限。

mysql> grant process,super on *.* to 'cacti'@'%' identified by 'cacti';
mysql> grant all privileges on cacti.*  to cacti@"%" identified by "cacti";
三，模板导入
在cacti管理界面（Import Templates）导入cacti_host_template_x_db_server_ht_0.8.6i.xml。

四，添加设备

创建Graph。在Console选项卡下的左侧菜单栏中选择Devices，为要监控的主机新建一个Devices或选择已有Devices。
在Associated Graph Templates中添加想要监控MySQL状态的Graph Templates（如X MySQL Connections GT模板）。
并点击最上面的Create Graphs for this Host链接，在Graph Templates的选择框中选择X MySQL Connections GT,然后点击Create按钮，出现以下WEB页。

```



