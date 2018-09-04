---


---

# systemd

[TOC]



## 核心概念 unit

​	unit由相关配置文件进行标识,识别和配置  主要包含了系统服务 监听socket 保存的快照和其他init信息,这些配置文件主要保存在:

- /usr/lib/systemd/system

  每个 服务最主要的启动脚本设置 ，类似于之前的/etc/init.d/

- /run/systemd/system

  系统执行过程中所产生的服务脚本，比上面目录优先运行

- /etc/systemd/system

  管理员建立的执行脚本，类似于 于/etc/rc.d/rcN.d/Sxx 类的功能，比上面目录优先运行

  ---

## unit常见类型

	service unit 文件扩展名.service 用于定义系统服务

	target unit 文件拓展名.target 用于模拟实现运行级别

	device unit 文件拓展名.device 定义内核识别的设备

	mount unit 文件拓展名.mount 定义文件系统挂载点

	socket unit 文件拓展名.socket 用于标识进程间通信用到的socket

	snapshot unit 文件拓展名.snapshot 管理系统快照

	swap unit 文件拓展名.swap 标识swap设备

	automount unit 文件拓展名,automount 文件系统自动挂载点

	path unit 文件拓展名.path 定义文件系统中的一文件或者目录
---

## 关键特性

​	基于socket的激活机制: socket与程序分离 

​	基于bus的激活机制 

​	基于device的激活机制

​	基于path的激活机制

​	系统快照 保存各个unit当前的状态信息在持久存储设备中

​	向后兼容sysv init脚本

**不兼容**

​	systemctl的命令是固定不变的

​	不是由systemd启动的服务,systemctl无法与之通信

---

## 管理系统服务

​	CentOS 7:service类型的unit文件来管控的

### 	systemctl命令:

​			---control the systemd system and service manager

systemctk [OPTION..] COMMEND [NAME...]

​				启动:
```bash
			service NAME start ====>systemctl start                                  NAME.service
```

​				停止:
```bash
			service NAME stop ====>systemctl stop NAM.service
```

​				重启:
```bash
			service NAME restart ===>systemctl restart 		                          NAM.service
```

​				状态:

```bash
			service NAME status====>systemd status NAM.service 
```

​				条件式重启:
```bash
			service NAME condrestart===>systemctl try-restart NAME.service
```

​				重载或重启服务:

```bash
			systemctl reload | restart NAME.service
```
​				重载或条件式重启服务

```bashbash
systemctl reload | try-restart NAME.service
```

查看当前服务激活状态与否

```bash
systemctl is-active NAME.service
```

查看所有已经激活的服务

```bash
systemctl list-units --type service
```

查看所有服务

```bash
chkconfig --list ====>systemctl list-units -t service --all | -a
```

设置开机自启动

```bash
chkconfig NAME on =====>systemctl enable NAME.service
```

禁止开机自启动

```bash
chkconfig NAME off =====>systemctl disable NAME.service
```

查看某服务是否开机自启动

```bash
chkconfig --list NAME =====>systemctl is-enabled NAME.service
```

禁止某服务设定开机自启动

```
systemctl mask NAME.service
```

取消禁止

```bash
systemctl unmask NAME.service
```

查看服务的依赖关系

```bash
systemctl list-dependencies NAME.service
```



---

### 管理target units

​	运行级别:

- 0====> runlevel0.target poweroff.target
- 1====> runlevel1.target rescue.target
- 2====> runlevel2.target multi-user.target
- 3====> runlevel3.target multi-user.target
- 4====> runlevel4.target multi-user.target
- 5====> runlevel5.target graphical-user.target
- 6====> runlevel6.target reboot.target

级别切换 

```bash
init N =====>systemctl isolate NAME.target
```

查看级别

```bash
runlevel =====>systemctl list-units --type target
```

查看所有级别

```bash
systemctl list-units -t target -a
```

获取默认运行级别

```bash
systemctl get-default
```

设定默认运行级别

```bash
systemctl set-default NAME.target
```

切换到紧急救援模式

```bash
systemctl rescue
```

切换到emergency模式

```bash
systemctl emergency
```

其他常用命令

```bash
systemctl halt | poweroff |reboot |suspend |hibernate |hybrid-sleep
```



---

## service unit file

 文件通常是由3个部分组成

​	[unit] 定义unit类型无关的通用选项,提供unit的描述信息 unit的行为和依赖关系

​	[service] 与特定类型的专用选项,此处为service类型

​	[install] 定义由"systemctl enables"和"systemctl disable"命令在实现服务启用或禁用时候用到$一些选项

### unit段常用选项:

- description:描述信息;意义性描述
- after 定义unit$启动次序,表示当前unit应该晚于哪些unit启动,其功能与before相反
- requies:依赖到其他unit,强依赖,被依赖的units无法激活时,当前unit无法激活
- wants 依赖到的其他unit 弱依赖
- confilcts:定义units间的冲突关系

### service段的常用选项

**type** 用于定义影响execstart和相关参数的功能unit进程启动类型

- simple
- forking
- oneshot
- dbus
- notify 类型与simple
- idle

**execstart** 指明unit要运行命令和脚本 execstartpre ExecStartPost

**Restart**

### install段常用选项

**ailas**

**RequiredBy** 被哪些units依赖

**WantedBy** 

> 重载生效
>
> systemctl daemon-reload

