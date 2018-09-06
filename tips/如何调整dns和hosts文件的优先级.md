如何调整dns和hosts文件的优先级

```
修改文件 /etc/nsswitch.conf文件

里面的
hosts:   files dns   //file指的就是/etc/hosts文件
```

