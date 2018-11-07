# zabbix监控JMX的时候 key无法取到值问题

在填key的时候 明明的复制下来的 , 然而会提示未注册 , 这个时候可能是引号问题 , 需要转义 , 比如

```
jmx["Catalina.type=GlobalRequestProcessor.name=\"http-nio-8080\"",errorCount]
```



