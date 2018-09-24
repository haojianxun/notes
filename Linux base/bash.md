# bash

```
echo $?   //最后一条命令的退出码
```

```
cmd1 && cmd2   //cmd1执行成功  则执行cmd2      cmd1执行失败    则不执行cmd2
cmd1 || cmd2   //cmd1执行成功   则不执行cmd2    cmd1执行失败    则执行cmd2

例子:
ping -c1 -w1 172.10.1.1 >>/dev/null && echo "host is up" || echo "host is down"
意思是ping 172.10.1.1 能ping通则打印 host is up  ping不通则打印 host is down
```

``` 
[ 22 == 33] && echo "不等于" || echo "等于"

数字的比较 -eq   //eq的意思为 equal
         -ge   //ge的意思是 g=greater  e=equal  大于等于  greater than or equal
         -gt   //t=than  大于  greater than
         -le  //l=less e=equal  less than or equal  小于等于
         -lt  // less than  小于
         -ne  //not equal  不等于
         
字符串测试
         -n "string"   //字符串是否为不空 n=nonzero 不空为真  要比较的字符串要加引号
         -z "string"   //字符串是否为空 z=zero  空为真  要比较的字符串要加引号
         ==
         =~ 左侧能够被右侧的panter
         
```

