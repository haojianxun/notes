# zabbix

[TOC]

zabbix的程序构成:

zabbix_server :服务端守护进程

zabbix_agentd agent端守护进程

zabbux_proxy:代理服务器



trigger 触发器

一个监控项可以有多个trigger 但是一个trigger只能关联到一个监控项

触发器表达式

```bash
{<server>:<key>.<function>(<parameter>)}<operator><constant>
#server:主机名称;
#key:主机上关系的相应监控项的key
#function:评估采集到的数据是否在合理范围内时所用到的函数,其评估过程可以根据采取的数据,当时时间和其他因素来进行评估;
#目前触发器支持的函数有avg,count,change,date,dayofweek,delta,diff,iregexp,last,max,min,nodata,now,sumd等
#parameter:函数参数,大多数函数可以接受秒数为其参数,而如果在数值参数之前使用'#'作为前缀,则表示最近几次的取值,入sum(300)表示最近300s的取值之和,而sun(#30),表示最近30次取值之和.
#此外avg,count,last,min max还支持使用第二个参数,用于完成时间限定,例如max(1h,7d)将返回最近7天的最大数
```

