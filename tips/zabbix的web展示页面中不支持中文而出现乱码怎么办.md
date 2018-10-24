# zabbix的web展示页面中不支持中文而出现乱码怎么办

在`/etc/httpd/conf.d/zabbix.conf` 中  可以看到在`/usr/share/zabbix/conf` 文件夹中  进入这个文件夹  再找到`fonts` 文件夹 , 会发现有tty类型的字体文件  , 之后我们可以把我们的字体直接扔在这个`fonts` 目录下 , 之后再重命名为之前默认的文件名称即可

```
mv myfont.tty graphfont.tty   //将我们仍进来的myfont.tty文件修改为原来默认的graphfont.tty文件名
```

