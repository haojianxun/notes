# [saltstack-01] saltstack安装

## 安装

### 1.安装阿里镜像源

```
yum install -y https://mirrors.aliyun.com/saltstack/yum/redhat/salt-repo-latest-2.el7.noarch.rpm
```

### 2.配置中的域名修改

仍以 Centos 7 为例，初始化rpm包生成的配置文件为:

```
/etc/yum.repos.d/salt-latest.repo
```

文件中的访问地址需要替换成镜像站的路径，执行命令：

```
sudo sed -i "s/repo.saltstack.com/mirrors.aliyun.com\/saltstack/g" /etc/yum.repos.d/salt-latest.repo
```
### 3.安装saltstack

#### 3.1安装saltstack-master和minion

```
yum install -y salt-master
```

```
yum install -y salt-minion
```

### 4.启动

```
systemctl start salt-master

systemctl start salt-minion
```

