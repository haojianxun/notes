# FastDFS安装配置文档

## 准备环境 

本文的系统：CentOS 7

实验环境的节点：

| hostname | role    | ip            |
| -------- | ------- | ------------- |
| node1    | tracker | 192.168.100.1 |
| node2    | storage | 192.168.100.2 |
| node3    | client  | 192.168.100.3 |



在各个节点/etc/hosts文件中添加以下内容：

```
192.168.100.1 node1
192.168.100.2 node2
192.168.100.3 node3
```

以下的信息都在node1中进行,可以用ansible批量操作,ansible需要基于ssh通信,所以还需要加入以下东西

```bash
ssh-keygen -t rsa -P''
cat .ssh/id_rsa.pub >>.ssh/authorized_keys
chmod go=  .ssh/anthorized_keys

scp -p .ssh/id_rsa .ssh/authorized_keys root@node2:/root/.ssh
scp -p .ssh/id_rsa .ssh/authorized_keys root@node3:/root/.ssh
```





## 安装FastDFS

新系统的话要安装一些必要的包比如基础开发包，gcc，gcc-c++，pcre，prec-devel，openssl等

```
yum  -y install zlib zlib-devel pcre pcre-devel gcc gcc-c++ openssl openssl-devel libevent libevent-devel perl unzip net-tools wget
```



安装FastDFS

```bash
#如果有git,则用git下载
git clone https://github.com/happyfish100/libfastcommon.git
cd fastdfs
./make.sh && make.sh install   #安装FastDFS的依赖包libfastcommon

git clone https://github.com/happyfish100/fastdfs.git
cd fastdfs
sh make.sh && sh make.sh install   #安装FastDFS


#如果没有git的话,可以用wget下载
wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz    
wget https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz 
wget https://github.com/happyfish100/fastdfs-nginx-module.git

#安装libfastcommon库
tar -xzvf  libfastcommon-1.0.7.tar.gz
cd libfastcommon-1.0.7
./make.sh
./make.sh install

#安装FastDFS
tar -xzvf  libfastcommon-1.0.7.tar.gz
cd libfastcommon-1.0.7
./make.sh
./make.sh install
```

## 修改配置文件

```
cd /etc/fdfs     #配置文件都在这个目录
cp storage.conf.sample  storage.conf
cp tracker.conf.sample tracker.conf
cp client.conf.sample client.conf
```

### 修改tracker.conf

```
#bind an address of this host
#empty for bind all address of this host 
bind_addr=       #可以指定ip地址,不写为空的话则是绑定这个机子的所有ip
base_path=/data/fastdfs     #这个是指定存储数据和日志的地方
```

### 修改storage.conf

```
group_name=group1
bind_addr=
base_path=
store_path_count=2   #定义有几块虚拟磁盘
store_path0=/data/fastdfs/dev0   #虚拟磁盘的路径
store_path1=/data/fastdfs/dev1
tracker_server=192.168.100.1:22122
```

### 启动

```
/etc/init.d/fdfs_storaged start
/etc/init.d/fdfs_tracker start
```



## 安装nginx拓展

```
#如果有git的话
git


wget https://github.com/happyfish100/fastdfs-nginx-module/archive/master.zip
unzip master.zip

进入到nginx
./configure --add-module=../fastdfs-nginx-module-master/src/
make && make install


修改nginx配置  在server中加入
        location ~/group1/M00 {
            root /FILE_PATH;
            ngx_fastdfs_module;
        }
        
之后把fastdfs包解压下中的文件夹conf中的http.conf和mime.types等其他配置文件拷贝到/etc/fdfs/下(把/etc/fdfs下没有的配置文件从conf中拷贝过来)
之后还需要把fastdfs-nginx-module安装目录中 src 目录下的mod_fastdfs.conf也拷贝到 /etc/fdfs 目录下


之后配置mod_fastdfs.conf

base_path=/opt/fastdfs_storage #保存日志目录
tracker_server=10.211.55.5:22122 #tracker服务器的IP地址以及端口号
storage_server_port=23000 #storage服务器的端口号
url_have_group_name = true #文件 url 中是否有 group 名
store_path0=/opt/fastdfs_storage_data # 存储路径
group_count = 1 #设置组的个数

组的设置如下
[group1]
group_name=group1
storage_server_port=23000
store_path_count=1
store_path0=
```

