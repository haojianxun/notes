# puppet-intro

[TOC]

## 学习puppet

相关书籍

- 《pro puppet》第一版和第二版，中文版叫《精通puppet配置管理工具》
- 《Puppet 2.7 Cookbook RAW》
- 《puppet实战》 刘宇



## 前期配置

说明

|              IP              | HOSTNAME |
| :--------------------------: | :------: |
| 192.168.100.110/24   [mater] |  master  |
| 192.168.100.111/24  [agent1] |  agent1  |
| 192.168.100.112/24  [agent2] |  agent2  |
| 192.168.100.113/24  [agent3] |  agent3  |

网关是192.168.100.110 , DNS服务器是192.168.100.110

### 设置主机名

```
vim /etc/sysconfig/network  #修改HOSTNAME的值 mater,agent1~3同理
```

### 设置ip地址

```
vim /etc/sysconfig/network-scripts/ifcfg-eth0  #mater agent1~3同理
```

### 关闭防火墙

```
iptables stop | iptables -F
```

### 关闭selinux

```
setenforce 0 #查看用getenforce
```

### 设置ssh-key互相通信

```
ssh-keygen

for i in {1..3}; do ssh-copy-id -i 192.168.100.11$i; done
```

### 修改host文件

```
vim /etc/hosts

for i in {1..3}; do scp /etc/hosts 192.168.100.11$i:/etc/; done
```

### 同步时间

```
chrony或者ntp设置
```

## 安装puppet

安装包---在master上面安装

```
yum -y install puppet puppet-server facter

查看配置文件
cp /etc/puppet/puppet.conf{,.bak} #备份配置文件
vim /etc/puppet/puppet.conf
[main]
    logdir = /var/log/puppet  #默认日志存放路径
    rundir = /var/run/puppet  #pid存放路径
    ssldir = $vardir/ssl #证书存放目录，默认$vardir为/var/lib/puppet
[agent]
    classfile = $vardir/classes.txt
    localconfig = $vardir/localconfig
    server = puppetmaster.kisspuppet.com #设置agent认证连接master端的服务器名称，注意这个名字必须能够被节点解析
    certname = puppetmaster_cert.kisspuppet.com #设置agent端certname名称
[master]
    certname = puppetmaster.kisspuppet.com  puppetmaster.kisspuppet.com #设置puppetmaster认证服务器名
```

在agent1~3上面安装包

```
yum -y install puppet facter

查看配置文件
cp /etc/puppet/puppet.conf{,.bak}
vim /etc/puppet/puppet.conf
[main]
    logdir = /var/log/puppet
    rundir = /var/run/puppet
    ssldir = $vardir/ssl

[agent]
    classfile = $vardir/classes.txt
    localconfig = $vardir/localconfig
    server = puppetmaster.kisspuppet.com  #指向puppetmaster端
    certname = agent1_cert.kisspuppet.com #设置自己的certname名
```

### 互相认证

```
[root@agent1 ~]# puppet agent --test #向master断发起认证

[root@master ~]# puppet cert --list --all #查看认证情况
[root@master ~]# puppet cert --sign agent1_cert.kisspuppet.com #注册agent1
[root@master ~]# puppet cert --list --all #再次查看认证情况


[root@master ~]# puppet agent --test #master自己申请agent认证
[root@master ~]# puppet cert --sign --all #注册所有请求的节点
[root@master ~]# puppet cert --list --all #查看所有节点认证
```

## puppet命令详解

使用`puppet help` 来查看命令帮助

### puppet语法

```
Usage : puppet <subcommand> [options] <action> [optins]

子目录的查看帮助语法是
'puppet help <subcommand> <action>' for help on a specific subcommand action
'puppet help <subcommand>'for help on a specific subcommand

查看所有可用的资源类型
puppet describe --list
```

### manifest

puppet的程序文件叫做manifest,以.pp作为文件后缀名

- puppet语言核心是'资源定义' , 定义一个资源核心就在于描述目标状态
- manifest实现了常见的程序逻辑,如条件语句,资源集合等
- manifest定义resource

"puppet apply"子命令能将一个manifest中描述的目标状态强制实现

### 资源定义

**syntax**

every resource has a **type** , a **title** , and a set of **attributes**

```
type {'title':
	attribute => value,
}
```

#### 示例

```
user {'puppet':
	ensure	=>present,
	gid		=>'666',
	uid		=>'666',
	shell	=>'/bin/bash',
	home	=>'/home/puppet',
	managehome	=>true,
}
```

注意:

- 在定义资源类型时必须使用小写字符,

- 资源名称仅是一个字符串,但是要求在同一个类型中必须唯一

  比如,可以用名字叫nginx的service资源和package资源,但是在package类型的资源中只能有一个名叫nginx

- `puppet resource` 命令可以交互式查找和修改puppet资源

### 资源类型

#### group

​	manage group

​	使用`puppet describe group` 命令来获取帮助

attribute:

​	name:组名,可以不写,不写的话就默认title是组名

​	gid:GID

​	system:是否是系统组

​	ensure:目标状态 present/absent

​	members:成员用户

#### user

​	manage users

​	使用`puppet describe user` 命令来获取帮助

attribute:

​	name:用户名,不写则title默认就是用户名

​	ensure:present/absent

​	uid

​	gid

​	groups:附加组,不能包含基本组

​	comment:注释

​	expiry:过期时间

​	home:家目录

#### package

​	manage packages

attribute:

​	ensure:installed , present , latest , absent

​	name:包名

​	source:程序包来源,一般是已经自己搞好的rpm或者dpkg

#### service

​	manage running services

attribute:

​	ensure:stopped(also called 'false') , running(also called 'true')

​	enable:true/false/manual

​	name:

​	path:

​	restart

#### file

manages files,including their content , ownership , and permissions

attribute:

​	ensure:present , absent , file , directory , link

​		file:普通文件,内容由content属性生成或者复制source属性文件路径来创建

​		link:符号链接文件,必须由target属性指明链接的目标文件

​		directory:目录 , 同通过source指向的路径复制生成,recurse属性指明是否递归复制

​	path:路径

​	source:源文件

​	content:文件内容

​	target:符号链接的目标文件

​	owner:属主

​	group:属组

​	mode:权限

​	atime/ctime/mtime:时间戳

示例:

```
file{'test.txt';
	path		=>'/tmp/test.txt',
	ensure		=>file,
	content		=>"hello world",
	owner		=>'nginx',
	mode		=>0664
	source		=>'/etc/fstab',
}
file{'test.symlink';
	path		=>'/tmp/test.symlink',
	ensure		=>link,
	target		=>'/tmp/test.txt',
	require		=>File['test.txt'],
}

```

#### exec

user/group:运行命令的用户身份

path

onlyif:指定一个命令,此命令执行成功,当前命令才能运行

unless:指定一个命令,此命令运行失败,当前命令才会运行

refresh:重新执行当前command的替代命令

refreshonly:仅接收到订阅的资源的通知才会运行

#### cron

command:要执行的任务

ensure:present/absent

hour:

minute:

monthday

month

weekday:

user

target:添加为哪个用户的任务

name:cron job的名称



#### notify

attribute:

​	message:信息内容

​	name:信息名称





特殊资源命令

resource special attributes

**Name/Namevar**

- most types have an attribute whith identifies a resource on the target system
- this is referred to as the 'namevar' , and is often simply called 'name'
- namevar values must be unique per resource type , with only rare exceotions(such as exec)

**ensure**

- ensure =>file 存在且为一个普通文件
- ensure =>directory 存在且为一个目录
- ensure =>present 存在,可通用于上述描述情况
- ensure=>absent 不存在

### 资源间的次序

puppet提供了before,require,notify,subscribe来定义资源间的相关性

资源的引用通过"Type['title']"的方式进行,如User['nginx']

比如

```
user{'redis':
	ensure		=>present,
	gid			=>800,
	uid			=>800,
	system		=>true,
}

group{'redis';
	ensure		=>present,
	gid			=>800,
	system		=>true,
	before		=>User['redis'],
}
```



### puppet variable

$variable_name=value

> - puppet的变量名称必须以$开头,赋值操作符为=
>
> - 任何正常数据类型的值都可以赋值给puppet中的变量
>
> - puppet的每个变量都有两个名称:简短名称和长格式完全限定名称(FQN),完全限定名称的格式为"$scope::variable"
>
>   > scope是一个特定代码区域,用于同程序中的其他代码隔离开来
>   >
>   > 在puppet中scope可用于限定变量和资源默认属性的作用范围,但是不能用于限定资源名称和资源引用的生效范围
>   >
>   > 如果要访问非当前scope中的变量,则需要通过完全限制名称进行,如$vhostdir=$apache::params::vhostdir
>   >
>   > top scope的名称为空,因此如果要引用其变量,则需要使用类似"$::osfamily"的方式进行

#### 数据类型

- 字符型:引号可有可无,单引号是强引用,双引号是弱引用
- 数值型:默认均识别为字符串,仅在数值上下文才以数值对待
- 数组:[]中以逗号分隔元素列表
- 布尔型值:true false,不加引号
- hash:{}中以逗号分隔k/v数据列表,键为字符型,值为任意puppet支持的类型,键值之间以"=>"分隔
- undef:未赋值型

#### 正则表达式

(?<ENABLED OPTION>:<PATTERN>)

(?-<DISABLED OPTION>:<PATTERN>)

​	OPTIONS

​		i:忽略字符大小写

​		m:把.当作换行符

​		x:忽略<PATTERN>中的空白字符

#### operators

comparison operators

- ==(equality)
- !=(non-equality)
- <(less than)
- \>(greater than)
- <=(less than or equal to)
- \>=(greater than or equal to)
- =~(regex match)
- !~(regex non-match)
- in

boolean operators

- and
- or
- !(not)

arithmetic operators

- +(addition)
- -(subtraction)
- /(division)
- *(multiplication)
- \>> (right shift)
- <<(left shift)

#### if语句

if CONDITIONS{

​	statement,

​	....

}

condition的给定方式

- 变量
- 比较表达式
- 有返回值的函数

比如

```
if $osfamily=~/(?-mx:debian)/{
	$webserver='apache2'
}else{
	$webserver='httpd'
}
```

#### case语句

case CONTROL_EXPRESSION{

​	case1:{....}

​	case2:{....}

​	....

​	default:{....}

}

control_expression

- 变量
- 表达式
- 有返回值的函数

各case的给定方式

- 直接字符串
- 变量
- 有返回值的函数
- 正则表达式模式
- default

比如

```
case $osfamily{
	"RedHat":{$webserver='httpd'}
	/(?i-mx:debian)/:{$webserver='apache2'}
	default:{$webserver='httpd'}
}
```

#### selector语句

CONTROL_VARIABLE?{

​	case1 =>value1,

​	case2 =>value2,

​	...

​	default =>valueN,

}

control_variable的给定方法

- 变量
- 有返回值的函数

各case的给定方法

- 直接字串
- 变量
- 有返回值的函数
- 正则表达式模式
- default

比如

```
$pkgname=$opratingsystem?{
	/(?i-mx:(ubuntu|debian))/   =>'apache2',
	/(?i-mx:(redhat|fedora|centos))/  =>'httpd',
	default						=>'httpd',
}
```

## puppet中的类

类:puppet中命名的代码模块,常用于定义一组通用的目标资源,可在puppet全局调用

类可以被继承,也可以包含子类

### 语法格式

```
class NAME{
	...puppet code...
}

class NAME(parameter1,parameter2){
	....puppet code....
}
```

类代码只有声明之后才会执行,调用方式

- include CLASS_NAME1,CLASS_NAME2,...

- class{'CLASS_NAME':

  ​	attribute =>value,

  }

示例

```
class apache2{
	$pkgname=$opratingsystem?{
	/(?i-mx:(ubuntu|debian))/   =>'apache2',
	/(?i-mx:(redhat|fedora|centos))/  =>'httpd',
	default						=>'httpd',
}

    package{"$webpkg":
        ensure =>installed,
    }

    file{'/etc/httpd/conf/httpd.conf':
        ensure		=>file,
        owner		=>root,
        group		=>root,
        source		=>'/tmp/httpd.conf',
        require		=>Package["$webpkg"],
        notify		=>Service['httpd'],
    }

    service{'httpd':
        ensure		=>running,
        enable		=>true,
    }
}

include apache2
```

示例2

```
class web($webserver='httpd'){
	package{"$webserver":
		ensure		=>installed,
		before		=>[File['httpd.conf'],Service['httpd']],
	}
	
	file{'httpd.conf':
		path		=>'/etc/httpd/conf/httpd.conf',
		source		=>'root/manifests/httpd.conf',
		ensure		=>file,
	}
	
	service{'httpd':
		ensure		=>running,
		enable		=>true,
		restart		=>'systemctl restart httpd.service',
		subscribe	=>File['httpd.conf'],
	}
}

class{'web':
	webserver	=>'apache2',
}
```

类继承的方式

```
class SUB_CLASS_NAME inherits PARENT_CLASS_NAME{
	.....puppet code...
}
```



在子类中为父类的资源新增属性或者覆盖指定的属性的值

```
Type['title']{:
	attribute		=>value,
	...
}

比如
class nginx::proxy inherits nginx {
	Service['nginx']{
		subscribe		=>File['nginx-proxy.conf'],
	}
```

## puppet模板

erb  模板语言 embedded ruby

puppet的erb语法

https://docs.puppet.com/puppet/latest/reference/lang_template_erb.html

文本文件中内嵌变量替换机制

`<%= @VARIABLE_NAME %>` 

```
file{'title':
	ensure		=>file,
	content		=>template('/PATH/TO/ERB_FILE'),
}

示例
file{'nginx.conf':
	path		=>'/etc/nginx/nginx.conf',
	ensure		=>file,
	content		=>template('/root/manifests/nginx.conf.erb'),
	require		=>Package['nginx'],
}
```

模块就是按照一个约定的预定义的结构存放了多个文件或子目录的目录,目录里的这些文件或者子目录必须遵照一定的格式命名规范

puppet会在配置的路径下查找所需要的模块

MODULES_NAME:

​	manifests/

​		init.pp

​	files/

​	templates/

​	lib/

​	spec/

​	tests/

模块名只能以小写字母开头,可以包含小写字母,数字和下划线,但是不能使用"main"和"settings"

manifests/

​	init.pp:必须一个类定义,类名称必须与模块名称相同

files/:静态文件

​	puppet URL

​		puppet:///modules/MODULE_NAME/FILE_NAME

templates/

​	template('MOD_NAME/TEMPLATE_FILE_NAM')

lib/:插件目录,常用于存储自定义的facts以及自定义类型

spec/ 类似于tests目录,存储lib/目录下插件的使用帮助和范例

tests/ 当前模块的使用帮助或使用范例文件



puppet config命令

​	获取或设定puppet配置参数

​		puppet config print [argument]

​		puppet查找模块文件的路径:modulepath

​	`puppet config print modulepath`  , 

可以指定自己要的路径,比如:

`puppet apply --modulepath=/root/dev/modules -e "include::server"` 

模块更多的语法可以用`puppet help module` 