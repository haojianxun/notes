## zookeeper安装

zookeeper官方地址: http://www.apache.org/dyn/closer.cgi/zookeeper/

## 主机准备

|       ip        |    name    |
| :-------------: | :--------: |
| 192.168.200.148 | zookeeper1 |
| 192.168.200.149 | zookeeper2 |



## 安装

```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

tar -xvzf zookeeper-3.4.10.tar.gz

mv zookeeper-3.4.10/ /usr/local/zookeeper
```

## 配置环境变量

```
vim /etc/profile

export PATH=/usr/local/zoo/bin:$PATH

source /etc/profile
```

## 创建数据目录

```
mkdir -p /data/zookeeper
```

## 修改配置文件

```
cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg

vim /usr/local/zookeeper/conf/zoo.cfg
dataDir=/data/zookeeper
server.1=192.168.200.148:2888:3888
server.2=192.168.200.149:2888:3888
```

在zookeeper1上执行

```
echo 1 > /data/zookeeper/myid
```

在zookeeper2执行

```
echo 2 > /data/zookeeper/myid
```

## 启动

```
cd /usr/local/zookeeper && ./bin/zkServer.sh start
```

