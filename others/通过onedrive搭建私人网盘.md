# 通过onedrive搭建私人网盘

具体项目地址: https://github.com/donwa/oneindex

我们可以用docker来运行它

```
docker run -d -p 8080:80 --name onedrive --restart=always yinaoxiong/oneindex
```

之后打开我们阿里云服务器的地址的8080端口就可以了(前提是把8080端口放开) 

之后进入安装页面



其中

获取程序的id和secret

地址: https://apps.dev.microsoft.com/#/appList



填入之后 就会跳转到登陆页面 , 登陆微软账号, 或者是office365账号(网上很多这种账号 里面onedrive空间是5T)

绑定即可 之后就搭建完成