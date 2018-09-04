# CentOS安装过程启动流程

boot.cat

stage2 isolinux/isolinux.bin

配置文件是isolinux/isolinux.cfg

- 对应的菜单选项

  加载内核 isolinux/vmlinux

  向内核传递参数 append initrd=initrd.img

装载根文件程序 加载anaconda

- 默认是图像界面 要是想启动文本的 就可以传递参数 text 方法是按esc键 
- boot:linux:method



anaconda配置

- 交互式配置
- 读取配置文件中事先定义好的配置项给出的 这个文件就是kickstart文件

安装引导选项

- boot

  ​	text:文本安装方式

  ​	method 手动指定使用的安装方式

  ​	与网络相关的引导选项

   		ip=IAPADDR

  ​		netmask=MASK

  ​		gateway=GW

  ​		dns=

  ​	远程访问概念

  ​		vnc

  ​		vncpassswd

  启动紧急救援模式

  ​		rescue

  装载额外驱动

  ​		dd

  ​







