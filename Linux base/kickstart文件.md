# kickstart文件

kickstart文件

脚本段

- %pre 安装前脚本

  运行环境:运行安装介质上的微型Linux系统环境

- %post安装后脚本

  运行环境:安装完成后的系统

命令段中的必备命令

- authconfig:认证配置方式

  ​	`authconfig --enableshadow --passalgo=sha512` 

- bootloader 定义bootloader安装位置和相关配置

  ​	`bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"` 

- keyboard 键盘配置

  `keyboard us`

- lang 语言类型

  `lang zh_CN.UTF-8` 

- part:分区布局

  `part /boot --fstype=ext4 --size=500` 

  `part pv.008002 --size=51200 `   

- rootpw 管理员密码

  ​	`rootpw --iscrypted ........` 

- timezone 时区

  `timezone Asia/Shanghai` 

- 其他

  clearpart 清除分区

  ​	`clearpart --none --drivevs=sda` 清空磁盘分区

  volgroup 创建卷组

  ​	`volgroup myvg --pesize=4096 pv.008002` 

  logvol 创建逻辑卷

  ​	`logvol /home --fstype=ext4 --name=lv_home --vgname=myvg --		   	            size  =51200` 

  生成加密密码方式

  ​	openssl passwd -1 -salt \` openssl rand -hex 4 \`

可选命令
- install OR upgrade 安装或者升级

- text 安装界面类型 text为TUI  默认是GUI

- network配置网络接口
   `network --onboot yes --device eth0 --bootproto dhcp --noipv6` 

- firewall 防火墙
   `firewall --disabled`

- selinux 安全访问linux
   seliunx --disabled

- halt 或者poweroff或者reboot:安装完成之后的行为

- repo 安装时使用的软件仓库

- url 安装的时候用repository  格式是url格式

- 系统安装完成之后禁用防火墙 
   CentOS 6:
   	`service iptables stop`
   	`chkconfig iptables off`
   CentOS 7:
   	`systenctl stop firewalld.service`
   	`systemctl disable firewalld.service`

- 系统安装完成禁用selinux

  编辑/etc/sysconfig/selinux或者/etc/selinux/config文件

  修改selinux的值为为Permissive|disabled

     立即生效
- 安装一个system-config-kickstart的包 可以打开图形窗口来配置kickstart
	检查语法错误 