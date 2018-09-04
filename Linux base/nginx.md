# nginx配置

[TOC]

## 配置文件路径

```bash
/etc/nginx/nginx.conf   或者 /etc/nginx/conf.d/*conf
#在/etc/nginx/nginx.conf的配置文件中有一句include /etc/nginx/conf.d/*.conf;   故有/etc/nginx/conf.d/*.conf
```

## 主配置文件结构

```bash
main block：主配置段，也即全局配置段；
	event {
						...
	}：事件驱动相关的配置；
	http {
		...
	}：http/https 协议相关的配置段；
	mail {
		...
	}
	stream {
		...
	}
#配置文件每行结尾要以;结束  它是类c风格的配置文件
```

### main配置段配置

```bash
分类
#正常运行必备的配置
#优化性能相关的配置
#用于调试及定位问题相关的配置
#事件驱动相关的配置


#正常运行必备的配置
user nginx; #nginx运行的用户
pid        /var/run/nginx.pid; #指定存储nginx主进程进程号码的文件路径
load_module modules/ngx_http_geoip_module.so; #载入额外的模块的
include file | mask; 载入额外配置文件的


#优化性能相关的配置
worker_processes number | auto;  #指定cpu的处理数目
worker_cpu_affinity cpumask ...;  cpu的优先级
	worker_cpu_affinity auto [cpumask];					
	CPU MASK：
		00000001：0号CPU
		00000010：1号CPU
worker_priority number #设置worker进程优先级 [-20,20]
worker_rlimit_nofile number #worker进程所能打开文件的上限

#调试定位
daemon on|off #是否以daemon守护进程运行nginx
master_process on| off 是否以master/worker模型来运行 默认on
error_log file [level] #

#事件驱动相关配置
events {
.....
}

worker_connections number # 每个worker所能打开的最大并发进程连接数 默认1024
use method # 指明并发连接所用的处理方法  比如 use epoll

accept_mutex on | off #处理新的连接请求的方法；on意味着由各worker轮流处理新请求，Off意味着每个新请						 求的到达都会通知所有的worker进程








```

