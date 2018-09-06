# rsync+inotify

[TOC]

inotify是Linux核心子系统之一，做为文件系统的附加功能，它可监控文件系统并将异动通知应用程序。本系统的出现替换了旧有Linux核心里，拥有类似功能之dnotify模块

优点
相较于被inotify替换较旧的 dnotify模块，inotify有诸多益处。[2][3]在旧的模块中，程序必须为每一个被监控的目录创建file descriptor，这种作法很容易让进程拥有的file descriptor逼近系统允许的上限，进而形成瓶颈。dnotify产生的file decriptor也会导致系统资源忙碌，使可移除设备无法被移除，徒增使用上的困扰。
由于dnotify只能让程序员监控目录层级的变化，“精细度”亦是“dnotify”的劣势之一。为此，程序员必须付出额外的心力，自行撰写代码以期追踪更细微的文件系统事件。
inotify相较之下使用较少的file descriptor，亦允许select()与poll()接口，优于dnotify使用的信号系统。这也使得inotify与既有以select()或poll()为基础之库(如：Glib)集成更加便利。



## 判断是否支持inotify

```Bash
uname -r #看看内核是否到达了2.6.13或者以上,如果低于2.6.13就需要重新编译
ll /proc/sys/fs/inotify/
```

安装inotify-tools

```
tar -xvf inotify-tools-3.14.tar.gz
cd inotify-tools-3.14
./configure --prefix=/usr/local/source/inotify-tools
make & make install
```

安装完成后，在安装目录的bin目录下， 有两个可执行程序inotifywait和inotifywatch

inotifywait：用于等待文件或目录上的特定事件，可以监控任何一组文件或目录，或者是监控整个目录树（目录、子目录、子目录的子目录等）。在shell脚本中使用的是inotifywait。 inotifywatch：用于收集被监视的文件系统的统计数据，包括每个inotify事件发生多少次等信息

# inotify监控事件

inotify 可以监视的文件系统事件包括：

```
access：文件被读取
modify：文件被修改。
attrib：文件属性被修改即元数据（Metadata），如执行chmod、chown、touch 等指令。
close_write：具有可写权限的文件被关闭。文件内容不一定修改。
close_nowrite：不可写文件被关闭，即只读文件被关闭。
open：文件被打开
moved_from：文件被移走。即文件从被监视的目录中移走。
moved_to，文件被移来。即文件被移动到被监视的目录下。
create：创建新文件
delete：文件被删除，如rm
delete_self：自删除，即一个可执行文件在执行时删除自己，删除后，文件不会被监控。
move_self：自移动。即一个可执行文件在执行时移动自己，移动后，该文件不会被监控。
unmount：被监控的文件或目录所在的文件系统被 umount，此后不会被监控。
close，文件被关闭，等同于(CLOSE_WRITE | CLOSE_NOWRITE)
move：文件被移动，等同于move_to和move_from
注：上面所说的文件也包括目录。
```

## rsync inotify实时备份

inotify的脚本配置

```
#!/bin/bash
INOTIFYWAIT="/usr/local/source/inotify-tools/bin/inotifywait"
EVENTS="-e create,delete,move,modify,attrib"
BACKUP="/backup/"
SECRETS_FILE="/etc/rsync/rsync.pass"
USER="felix"
HOST="rsync.felix.com"
MODULE="backup"
$INOTIFYWAIT -mrq --timefmt '%F %T' --format '%T %w%f' $EVENTS $BACKUP | while read line
do
        rsync -vzrogpt --delete --progress --password-file=$SECRETS_FILE $BACKUP $USER@$HOST::$MODULE &> /dev/null
done
```

创建/backup目录

```
[root@vm2 ~]# mkdir /backup
[root@vm2 ~]# ls -l /backup/
total 0
[root@vm2 ~]# 
```

说明：这里是将所有需要备份的内容放在/backup目录下面，在该目录下面的所有内容都会自动备份。

启动脚本

```
[root@vm2 ~]# sh /data/script/inotify.sh &
```

测试

```
[root@vm2 ~]# cp /etc/hosts .
[root@vm2 ~]# ls -l 
```

同一台机器上2个目录保存同步

```
#!/bin/bash

INOTIFYWAIT="/usr/local/source/inotify-tools/bin/inotifywait"
EVENTS="-e create,delete,move,modify,attrib"
SOURCE="/dir1/"
DEST="/dir2/"
$INOTIFYWAIT -mrq --timefmt '%F %T' --format '%T %w%f' $EVENTS $SOURCE | while read line
do
        rsync -vzrogpt --delete --progress $SOURCE $DEST &> /dev/null
done
```

创建测试目录

```
[root@vm2 script]# mkdir /dir1
[root@vm2 script]# mkdir /dir2
[root@vm2 script]# 
```

启动脚本

```
sh /data/script/inotify_local_dir.sh &
```

测试

```
[root@vm2 ~]# ls -l /dir1 /dir2
/dir1:
total 0

/dir2:
total 0
[root@vm2 ~]# cp /etc/hosts /dir1
[root@vm2 ~]# ls -l /dir1 /dir2
```