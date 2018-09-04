# openssl

[TOC]

命令分类

- 标准命令
- 消息摘要命令dgst
- 加密命令enc


## enc命令

​	enc  ---  symmetric cipher routines 对称加密程序

> 对称加密:
>
> 加密和解密使用相同的密钥,特点是计算量小,算法公开,加密速度快,不足之处是安全性得不到保障,具体算法有:3DES,TDEA,Blowfish,RC5,IDEA,容易被破解

### 加密

```bash
openssl enc -e -des3 -a -salt -in testfile -out testfile.cipher
```

### 解释:

> enc : enc是openssl下的子命令
>
> -e  encrypt the input data: this is the default.把输入的数据加密  e为encrypt 					的缩写
>
> -des3 : 3des算法 3des算法是des算法的变种,原来的des算法容易被暴力破解,3des即是对每个数据块进行3次加密,通过增加des的密钥长度增加破解难度
>
> -a  :base64 process the data. This means that if encryption is taking place the data is base64 encoded after encryption. If decryption is set then the input data is base64 decoded before being decrypted
>
> -salt :加盐  
>
> -in filname :输入的文件名  要加密的文件
>
> -out filname : 输入的文件名 

### 解密

```bash
openssl enc -d -des3 -a -salt –in testfile.cipher -out testfile
```

> -d : 解密 (decrypt the input data)

### 生成用户密码

```bash
openssl passwd -1 -salt SALT( 最多8 位)
```

### 生成随机数

```bash
openssl rand -base64|-hex NUM
```

> NUM:  表示字节数；-hex 时，每个字符4 位，出现的字
> 符数为NUM*2

### 生成私钥

```bash
openssl genrsa -out  /PATH/TO/PRIVATEKEY.FILE NUM_BITS
```

> genrsa - generate an RSA private key  ( generate:生成,)

### 从私钥中生存公钥

```bash
openssl rsa -in  PRIVATEKEYFILE  –pubout –out  PUBLICKEYFILE
```

证书申请及签署步骤：

1. 生成申请请求
2. 核验
3. 签署
4. 获取证书

## 创建CA 和申请证书

### 创建私有CA：

> openssl 的配置文件：/etc/pki/tls/openssl.cnf
>
> 里面的配置文件在[ CA_default ]中有详细介绍

1. #### 创建所需要的文件

   ```bash
   touch /etc/pki/CA/index.txt
   ```

   ```bash
   echo 01 > /etc/pki/CA/serial
   ```

2. #### CA自签证书

   ```bash
   cd /etc/pki/CA/
   (umask 066; openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)
   ```

## 生成自签名证书

```bash
openssl req -new -x509 –key /etc/pki/CA/private/cakey.pem -days 7300
```
   > -out /etc/pki/CA/cacert.pem
   > -new:  生成新证书 签署请求
   > -x509:  专用于CA 生成自 签证书
   > -key:  生成请求时用到的私 钥文件
   > -days n ：证书 的有效期限
   > -out / PATH/TO/SOMECERTFILE :  证书的保存路径

### 颁发证书

1. ##### 生成证书请求

   给web服务器生成私钥

   ```bash
   (umask 066; openssl genrsa -out/etc/http/ssl/http.key 2048
   ```

   生成证书申请文件

   ```bash
   openssl req -new -key /etc/httpd/ssl/httpd.key -days 365 -out /etc/httpd/ssl/httpd.csr
   ```

   将证书请求文件传输给CA

   CA 签署证书，并将证书颁发给请求者

   ```bash
   openssl ca -in /tmp/httpd.csr –out /etc/pki/CA/certs/httpd.crt -days 365
   ```
   查看证书中的信息

   ```bash
   openssl x509 -in  /PATH/FROM/CERT_FILE  -noout -text|subject|serial|dates
   ```


### 吊销证书

1.    在客户端获取要吊销的证书的serial

      ```bash
      openssl x509 -in / PATH/FROM/CERT_FILE -noout -serial -subject
      ```

2.    在CA 上，根据客户提交的serial 与subject 信息，对比检验是否与index.txt 文件中的信息一致

                                                   吊销证书

      ```bash
                                                   openssl ca -revoke /etc/pki/CA/newcerts/SERIAL .pem
      ```

3.    生成吊销证书的编号( 第一次吊销一个证书时才需要执行)

      ```bash
                                             echo 01 > /etc/pki/CA/crlnumber
      ```

4.    更新证书吊销列表

      ```bash
                                       openssl ca -gencrl -out /etc/pki/CA/crl/ca.crl
      ```

5.    查看crl 文件

      ```bash
                                    openssl crl -in /etc/pki/CA/crl/ca.crl -noout -text
      ```

## 概念和相关英文单词:

> **Certificate Authority (CA)**

>  **Certificate Revocation List (CRL)**

> **enc  Encoding with Ciphers.**

> **Certificate Signing Request (CSR)**

> **X.509** - 这是一种证书标准,主要定义了证书中应该包含哪些内容.其详情可以参考RFC5280,SSL使用的就是这种证书标准.

> **SSL** - Secure Sockets Layer,现在是改叫Transport Layer Security ---**TLS**

> **PEM** - Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码.
> **DER** - Distinguished Encoding Rules,打开看是二进制格式,不可读.

>  **CRT**  为certificate的三个字母

 > **CER** - 还是certificate,还是证书
 >
 > .pem的证书为测试证书,里面包含了公钥私钥请求等等信息
 >
 > 自己要想搞个证书,可以在/etc/pki/tls/certs/下    里面有个makefile文件 里面详细介绍了各种证书的生成  用make命令生成证书,生成不同用途的证书,后缀也要不同.比如.key .pem .crt .csr等等












