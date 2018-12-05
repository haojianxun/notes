#codis集群高可用搭建

## 安装go

```
wget https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
at << EOF  >>/etc/profile
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export GOPATH=/usr/local/go/work
path=$PATH:$HOME/bin:$GOROOT/bin:$GOPATH/bin
EOF
source /etc/profile
```

## 安装zookeeper

```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

tar -xvzf zookeeper-3.4.10.tar.gz

mv zookeeper-3.4.10/ /usr/local/zookeeper
```

### 配置zookeeper的环境变量

```
vim /etc/profile

export PATH=/usr/local/zoo/bin:$PATH

source /etc/profile
```

### 创建zookeeper数据目录

```
mkdir -p /data/zookeeper
```

修改zookeeper配置文件

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

### 启动

```
cd /usr/local/zookeeper && ./bin/zkServer.sh start
```



## 安装codis

```
mkdir -p $GOPATH/src/github.com/CodisLabs
cd $_ && git clone https://github.com/CodisLabs/codis.git -b release3.2
cd $GOPATH/src/github.com/CodisLabs/codis
make
```

### 配置环境 变量

```
# vim /etc/profile
export PATH=$GOPATH/src/github.com/CodisLabs/codis/bin:$PATH

# source /etc/profile
```

### codis配置

进入codis目录

```
cd $GOPATH/src/github.com/CodisLabs/codis
```

#### 配置codis-dashboard

```
# vim config/dashboard.toml
coordinator_name = "zookeeper"
coordinator_addr = "10.10.10.21:2181,10.10.10.22:2181,10.10.10.23:2181"
product_name = "codis-demo"
product_auth = "123456"
```

#### codis proxy配置

```
# vim config/proxy.toml
product_name = "codis-demo"
product_auth = "123456"
jodis_name = "zookeeper"
jodis_addr = "192.168.200.148:2181,192.168.200.149:2181,192.168.200.150"
```

#### codis server配置

```
#修改主配置
cp config/redis.conf config/redis-6379.conf
vim config/redis-6379.conf
bind IPADDR
dir /data/redis-data/redis-6379
....(包括一些密码相关的安全)


#修改从配置
cp redis-6379.conf redis-6380.conf
sed -i "s/6379/6380/g" redis-6380.conf
vim redis-6380.conf
....
slaveof <MASTERIP> <MASTERPORT>
...
```

```
mkdir -pv /data/redis-data/redis-{6379,6380}
```

#### codis fe配置启动

```
# vim config/codis.json
[
    {
        "name": "codis-demo",
        "dashboard": "127.0.0.1:18080"
    }
]
```

## Redis-sentinel

```
cp -fr $GOPATH/github.com/CodisLabs/codis/extern/redis-3.2.8/src/redis-sentinel /usr/local/codis/bin/

cp -fr $GOPATH/github.com/CodisLabs/codis/extern/redis-3.2.8/sentinel.conf /usr/local/codis/conf/

cd /usr/local/codis/ 
vim ./conf/sentinel.conf

bind 0.0.0.0 
protected-mode no 
port 26379 
dir “/usr/local/codis/data/
```



## 启动

codis集群中: 1个zookeeper（或1个zookeeper集群） + 1个codis dashboard + n个codis proxy + n个codis server + 1个Codis FE（可选） , 启动时候根据情况来启动 , dashboard启动的时候就启动一个就可以了

### codis-server启动

```
./bin/codis-server ./config/redis-6379.conf
./bin/codis-server ./config/redis-6380.conf
```

### codis-proxy启动

```
nohup ./bin/codis-proxy --ncpu=4 --config=config/proxy.toml --log=logs/proxy.log --log-level=WARN & 
```

其中cpu的最大值依据自己电脑来

### codis-dashboard启动

```
nohup ./bin/codis-dashboard --ncpu=4 --config=config/dashboard.toml --log=logs/dashboard.log --log-level=WARN &
```

### codis-fe启动

```
nohup ./bin/codis-fe --ncpu=4 --log=logs/fe.log --log-level=WARN --dashboard-list=config/codis.json --listen=0.0.0.0:8080 &
```

### sentinel启动

```
cd /usr/local/codis/bin/ 
nohup ./redis-sentinel ../conf/sentinel.conf &
```





## codis配置

登陆codis fe，http://192.168.200.148:8080

