```
修改backend中 balance的选项
vim /etc/haproxy/haproxy.cfg #编辑haproxy配置文件
backend web
	balance uri
	hash-type consistent
	server web1 172.16.0.1 check
	
systemctl reload haproxy.service

cd /var/www/html #切换到之前定义的web1主机172.16.0.1
for i in {1..20};do echo "test page $i @ BE1"> ./test$i.html;done #生成20个测试页面
之后用curl测试 可以发现是同一个url始终是在同一个服务器
```

