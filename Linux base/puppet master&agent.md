# puppet master/agent

使用`puppet help agent` 来获取帮助

在客户端怎么指明服务器端

```
puppet agent --server SERBERNAME --no-daemonize --noop -v
# no-daemonize  不启用守护进程 
#--noop 不是真的跑 , 不从服务器端取货运行代码 只是作为测试
#-v 显示详细信息
```

使用`puppet help cert` 来获取使用管理证书的命令

初始化master

`puppet master --no-daemonize --verbose` 

生成一个完整的配置参数列表

`puppet master --genconfig` 

`puppet agent --genconfig` 

打印基于默认配置生效的各配置参数列表

` puppet config <action> [--section SETION_NAME]` 

`puppet config print` 

基于命令行设置某参数的值

`puppet config set` 



master端管理证书签署

```
puppet cert <action> [--all][<host>]
	action
		list
		sign
		revoke
		clean
```

清单文件导入

`import '/PATH/FROM/SOME_MINIFEST_FILE'` 

## 站点清单的定义

主机名定义

​	主机名(主机角色)#-机架-机房-运营商-区域.域名

​		www1-rack1-yz-unicom-bj.magedu.com

```
/etc/puppet/manifests/site.pp
	node'base'{
		include ntp
	}
	
	node 'HOSTNAME'{
		....puppet code...
	}
	
	node /PATTERN/ {
		...puppet code...
	}
```

节点的继承

```
node NODE inherits PAR_NODE_DEF{
	....puppet code...
}
```

各环境配置

/etc/puppet/environments/{production,development,testing}

master支持多环境

```
[master]
#modulepath=
#manifest=
environment=production,development,testing

[production]
modulepath=/etc/puppet/environments/production/modules/
manifest=/etc/puppet/environments/production.manifests/site.pp

[development]
modulepath=/etc/puppet/environments/development/modules/
manifest=/etc/puppet/environments/development/manifests/site.pp

[testing]
modulepath=/etc/puppet/environments/testing/modules/
manifest=/etc/puppet/environments/testing/manifests/site.pp
```

在puppet3.6之后的版本配置多环境的方法

master支持多环境

配置文件puppet.conf

[master]

environmentpath=$confdir/environments



在多环境配置目录下为每个环境准备一个子目录

ENVIRONMENT_NAME/

​	manifests/

​		site.pp

​	modules/



agent端

[agent]

environment={production|development|testing}

额外配置文件

​	文件系统:fileserver.conf

​	认证(URL):auth.conf

## puppet kick

agent:

​	puppet.conf

​	[agent]

​	listen=true

​	auth.conf

​	path/run

​	method save

​	auth any

​	allow master.magedu.com

​	



​	path/

​	auth any



master端

​	puppet kick

​		puppet kick [--host\<HSOT\>]\[--all\] 

