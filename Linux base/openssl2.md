# openssl/ssl/加密/

[TOC]



## ssl会话三部

1. ### 客户端向服务器端索要并验证证书

2. 双方协商生成会话密钥

3. 双方采用会话密钥进行加密通讯

### 第一阶段clienthello

1. 支持的协议版本.比如tls1.2
2. 客户端生成一个随机数 稍后用户生成会话密钥
3. 支持的加密算法 比如aes 3des rsa
4. 支持的压缩算法

第二阶段 serverhello

1. 确认使用的加密通信协议版本 
2. 服务器端生成一个随机数 稍后用于会话密钥
3. 确认使用的加密方法
4. 服务器证书

### 第三阶段

1. 验证服务器证书  确认无误取出公钥
2. 发送信息给服务器
   - 一个随机数
   - 编码变更通知
   - 客户端握手结束通知

### 第四阶段

1. 收到客户端发来的三个随机数per-master-key后 计算生成本次会话所用到的会话密钥
2. 向服务器发送通知 确定加密方法 带我去握手结束通知

[pki][https://zh.wikipedia.org/wiki/%E5%85%AC%E9%96%8B%E9%87%91%E9%91%B0%E5%9F%BA%E7%A4%8E%E5%BB%BA%E8%A8%AD] Public Key Infrastructure 公开密钥基础设施

- 签证机构 CA
- 注册机构 RA
- 证书吊销列表 CRL
- 证书存钱库

TLS    Transport Layer Security   **传输层安全协议**  前身是**安全套接层**（Secure Sockets Layer，缩写：**SSL**）

## openssl

分为3种

- 标准命令
- 消息摘要命令
- 加密命令

### 对称加密

​	工具 openssl enc gpg 

​	支持是算法 3des,aes,blowflsh,towflsh

### 	enc命令

> -e encrypt 加密
>
> -d decrypt 解密
>
> -in 要加密的文件名称
>
> -out  加密后的文件名称

单向加密

​	工具 openssl dgst md5sum sha1sum sha224sum,...

​	dgst 命令  digest  

​		比如 

```
openssl dgst -md5 FILENAME
```

### 生成用户密码

openssl passwd

```
openssl passwd -1 -salt SALT
```

### 生成随机数

​	工具 rand

```
openssl rand -base64 NUM    //NUM是代表生成几位随机数
```

### 公钥加密

- 加密解密   算法:RSA ELGamal
- 数字签名  算法:RSA DSA ELGamal  工具:openssl rsautl gpg
- 密钥交换 算法: DH

### 生成密钥

```
生成私钥(umask 077;openssl genrsa -out /PATH NUM_BITS)
		在括号中表示启动子shell 077表示仅自己可读
提出公钥 openssl rsa -in /FILE -pubout //pubout 表示提出公钥
```

### Linux系统上的随机数生成器:

- /dev/random 仅从熵池中返回随机数  随机数用尽 阻塞

- /dev/urandom 从熵池返回 随机数用尽 用软件生成 非阻塞,伪随机,不安全

- 熵池的随机数的来源:

  硬盘io中断时间间隔

  键盘IO中断时间间隔

## CA

​	公共信任CA 私有CA

​	建立私有CA

​		openssl

​		OpenCA

### openssl命令

​	配置文件 /etc/pki/tls/openssl.cnf

```bash
####################################################################
[ ca ]
default_ca      = CA_default            # The default ca section  
# 上面是[ ca ]代表的是ca命令 下面就是ca命令的配置 默认的ca命令配置段是CA_default 即下面的部分
####################################################################
[ CA_default ]    #这个就是ca命令的默认配置段

dir             = /etc/pki/CA           # Where everything is kept  #ca命令的主目录
certs           = $dir/certs            # Where the issued certs are kept #certs存放位置
crl_dir         = $dir/crl              # Where the issued crl are kept 
#crl_dir 是crl的存放目录 crl就是certificate request list 证书请求列表的意思
database        = $dir/index.txt        # database index file.
#这个是存放数据量的  每个签证ca的索引
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/cacert.pem       # The CA certificate
#这个是代表ca的自签名证书  ca的自签名证书是cacert.pem
serial          = $dir/serial           # The current serial number
#这个的意思是说每签名的证书都会有个编号,序列号 每签完一个就加1
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key
#这个的意思是自己的私钥存放位置 而且还规定了自己的私钥文件名称
RANDFILE        = $dir/private/.rand    # private random number file
```

### 构建私有CA

​	在确定配置为CA的服务器上生成一个自签证书 并提供所需文件和目录即可

​	先创建需要的文件

```bash
touch /etc/pki/CA/index.txt
echo 01 > /etc/pki/CA/serial
mkdir -pv /etc/pki/CA/{certs,crl,newcerts} #有的话就不用创建了
```

步骤: 

1. 生成私钥

   ```bash
   cd /etc/pki/CA/private  #这个是私钥的存放位置 在配置文件中已经定义
   (umask 077;openssl genrsa -out /etc/pki/CA/private/cakey.pem 1024) #1024可改,需为2的n次方
   ```

2. 生成自签证书

   ```bash
   openssl req -new -x509 –key /etc/pki/CA/private/cakey.pem -days 7300
   -out /etc/pki/CA/cacert.pem
   -new:  #生成新证书 签署请求
   -x509:  #专用于CA 生成自 签证书, 自签的时候用
   -key:  #生成请求时用到的私 钥文件
   -days n ：#证书 的有效期限
   -out / PATH/TO/SOMECERTFILE : #保存路径
   ```

### 要用到证书进行安全通信的服务器,需要向CA请求签署证书

1.   在需要使用证书的主机生成证书请求;

   给web 服务器生成私钥

   ```bash
   (umask 066; openssl genrsa -out /etc/httpd/ssl/httpd.key 2048)
   ```

   生成证书申请文件

   ```bash
   openssl req -new -key /etc/httpd/ssl/httpd.key -days 365 -out /etc/httpd/ssl/httpd.csr
   ```

2. 将证书请求文件传输给CA

3. CA 签署证书，并将证书颁发给请求者

   ```bash
   openssl ca -in /tmp/httpd.csr –out /etc/pki/CA/certs/httpd.crt -days 365
   #注意：默认国家省 ，省  ，公司名称必须和CA 一致
   ```

4. 查看 证书中的信息

   ```bash
   openssl x509 -in  /PATH/FROM/CERT_FILE  -noout -text|subject|serial|dates
   ```

   ​





