# tcpdump nc nmap

```
tcpdump -i eth0  //指定eth0
tcpdump -i eth0 tcp port 80 -nn  //-nn表示  第一个n代表ip地址转为数字 第二个n是端口转为数字格式 表示目标地址或者源地址的80端口
tcpdump -i eth0 tcp dst port 80 -nn  //表示目标地址的80端口
```

```
tcpdump
	-X : 十六进制和ascii码
	-A: ASCII
```

```
nc   netcat的简称
还有一个nc是ncat 由nmap提供

nc -l 1234 > /tmp/fstab  //监听在本机1234端口 之后写文件到/tmp/fstab
nc 192.168.200.140 1234 < /etc/fatab  //往192.168.200.140的1234端口发文件 文件是/etc/fstab

nc -l 1234 </tmp/fstab  //监听在本机1234端口 传送文件/tmp/fsab
nc 192.168.200.140 1234 > /etc/fatab  //接收192.168.200.140的1234端口的文件 文件是/etc/fstab
```



#### 端口扫描

```
nc -v -w 1 192.168.200.140 -z 1-1000 //-v 表示详细信息 -w表示timeout时常 -z表示扫描端口范围
```

#### 简单聊天工具

```
在192.168.200.139上
nc -l 2233

在192.168.200.140上
nc 192.168.200.139 -p 2233
```

#### 用nc命令操作memcached

1）存储数据：`printf"set key 0 10 6rnresultrn" |nc 192.168.2.34 11211`

2）获取数据：`printf"get keyrn" |nc 192.168.2.34 11211`

3）删除数据：`printf"delete keyrn" |nc 192.168.2.34 11211`

4）查看状态：`printf"statsrn" |nc 192.168.2.34 11211`

#### 作为浏览器

```
echo -n "GET / HTTP/1.0"r"n"r"n" | nc host.example.com 80   
```

