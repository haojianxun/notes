iptables总结

1. 问:包是如何拆分的

​        答:报文是从应用层开始的,然后根据7层协议一层一层封装,到达目标主机以后再从底层向上一层一层拆分,其中就有tcp首部和ip首部

2. tcp首部和ip首部是什么

   见图片

3. iptables是从ipfirewall framework框架中调用的  防火墙是在内核实现的

   之后发展到了ipchains

   netfilter是工作在内核中

4. 防火墙在不同的地方设置了不同的规则  有5种  叫做 input output forward  prerouting postrouting

5. 表   filter表:过滤 防火墙

   net:地址转化的

   mangle 拆解报文 修改 再封装

   raw 关闭net表上的了解追踪

6. iptables的

