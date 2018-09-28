# 用entrypoint脚本来初始化docker自定义配置文件-以nginx为例

先创建一个文件夹 , 用来放我们实验需要的文件

```
mkdir test
```

进入文件夹

```
cd test/
```

先编辑dockerfile文件

vim Dockerfile

```dockerfile
FROM nginx:1.14-alpine
LABEL maintainer="hha"
ENV NGX_DOC_ROOT="/data/web/html/"
ADD index.html ${NGX_DOC_ROOT}
ADD entrypoint.sh /bin/
EXPOSE 80/tcp
CMD ["usr/sbin/nginx","-g","daemon off;"]
ENTRYPOINT ["/bin/entrypoint.sh"]
```

vim entryponit.sh

```shell
#/bin/sh
cat > /etc/nginx/conf.d/www.conf << EOF
server {
    server_name ${HOSTNAME};
    listen ${IP:-0.0.0.0}:${PORT:-80};
    root ${NGX_DOC_ROOT:-/usr/share/nginx/html};
}
EOF

exec "$@"
```

vim index.html

```html
<h1>TEST ENTRYPONIT NGINX</h1>
```

之后运行build构建docker

```
docker build -t mynginx:v1-1 .
```

运行这个docker

```
docker run --name myweb1 --rm -P mynginx:v1-1
```



查看刚刚起的docker ,看是否是我们自定义的那个配置文件

```
docker exec -it myweb1 /bin/sh

cat /etc/nginx/conf.d/www.conf  //看看这个配置文件是不是我们自定义的那个

ss -tnl   //可以看到80端口也监听了

wget -o - -q localhost  //默认的nginx主页
wget -o - -q DOCKERHOST  //访问用docker的hostname这个值的做的虚拟主机的主页
```

