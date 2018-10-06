# kubernetes如何创建被认证的用户

用户持有者的名称就是账号名 , 就是证书的subj

用来认证接入api-server

### 创建kubernetes认证的新用户test

```
cd /etc/kubernetes/pki
```

```
(umask 077; openssl genrsa -out test.key 2048)
```

```
openssl req -new -key test.key -out test.csr -subj "/CN=test"
```

```
openssl x509 -req -in test.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out test.crt -days 365
```

**查看刚刚创建的证书内容**

```
openssl x509 -in test.crt -text -noout
```

  

```
kubectl config set-credentials test --client-certificate=./test.crt --client-key=./test.key --embed-certs=true
```

**完成之后查看这个config**

```
kubectl config view
```



```
kubectl config set-context test@kubernetes --cluster=kubernetes --user=test
```

*注意:上述 `test@kubernetes` 中 test是用户名 kubernetes是集群名称*  , 指定用哪个用户访问哪个集群







**切换到test用户**

```
kubectl config use-context test@kubernetes
```



