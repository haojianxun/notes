# FastDFS调研

## FastDFS概述

作者余庆,网名是happy_fish100,作者的github地址是https://github.com/happyfish100,上面安装包和详细的介绍都有,还有nginx的拓展,其发表是传世贴好像在china unix论坛上,论坛上FastDFS板块的版主就是他(最早是帖子就在这里了,不过作者最后登陆时间是几年前...),作品是发表于2008年7月,第一版是v1.00



FastDFS是一款开源的轻量级分布式文件系统

- 纯C实现，支持Linux、FreeBSD等UNIX系统
- 类google FS，不是通用的文件系统，只能通过专有API访问，目前提供了C、Java和PHP API
- 适合中小文件（4KB< file_size < 500MB）
- FastDFS可以看做是基于文件的key value pair存储系统，称作分布式文件存储服务更为合适

## FastDFS架构

共有两个角色(严格是三个,其实还有client)

- **Tracker Server**：跟踪服务器，主要做调度工作，在访问上起负载均衡的作用。在内存中记录集群中group和storage server的状态信息，是连接Client和Storage server的枢纽。 因为相关信息全部在内存中，Tracker server的性能非常高，一个较大的集群（比如上百个group）中有3台就足够了。每个storage server会周期的向tracker发心跳信息,
- **Storage Server**：存储服务器，文件和文件属性（meta data）都保存到存储服务器上。以group为单位进行组织,任何一个storage server都应该属于某个group.一个group应该包含多个storage server,在同一个group内部,各storage server的数据互相冗余,每个数据存储目录创建2级子目录，每级265，总共65535个目录，新文件会以hash的方式路由到其中一个子目录下，然后将文件存储下来

## 功能

- upload：上传普通文件，包括主文件
- upload_appender：上传appender文件，后续可以对其进行append操作
- upload_slave：上传从文件
- download：下载文件
- delete：删除文件
- append：在已有文件后追加内容
- set_metadata：设置文件附加属性
- get_metadata：获取文件附加属性

## 实现过程

### Upload File

1. client发起请求
2. tracker找到合适的storage server
3. 找到storage server之后,将IP:PORT发给client
4. 上传文件(属性信息和内容)
5. 生成文件fid
6. 同步 存储文件信息到同组的其他节点

> FQA（以下都源自配置文件tracker.conf & storage.conf里面的具体配置）
>
> - **tracker如何挑选组**
>
>   具体参数是配置文件中的store_lookup
>
>   1. rr(轮询)
>   2. 指定组   
>   3. 基于可用空间进行均衡
>
> - **如何在组中挑选storage server**
>
>   1. rr
>   2. 以ip为次序,找第一个
>   3. 以优先级为次序,找第一个，具体参数upload_priority
>
> - **如何选择磁盘**
>
>   1. rr
>   2. 剩余可用空间大的优先
>
> - **生成FID**
>
>   由源头storage server ip , 创建时间戳,大小,文件校验码和一个随机数进行hash计算生成
>
>   具体格式如：group1/M00/path0/path1/hash  其中group1为组名，M00是虚拟磁盘名称，在storage.conf配置文件中由store_path配置， path0/path1是二级子目录  hash是系统自己生产的文件名 
>
> - 文件同步（之后具体细说）
>
>   每个storage server在文件存储完成后,会将其存储到binlog,binlog是不包含数据的元数据,binlog可用于同步

### Download File

1. client向tracker发请求
2. tracker根据文件名定位到group.并返回组内某个storage server的ip:port给client
3. client向得到的ip:port发请求
4. storage server查找文件,并返回给client

