# mysqldump中--databases选项的作用

```
mysqldump --databases hellodb --single-transaction --trigger- E > /root/hello-$(date +%F)
这样加了--databases的语句  在备份的时候先是会创建表的,也就是说备份了整个hellodb的  如果不加--databses的话,表明只是备份hellodb中的数据表,而在还原的时候不会在创建hellodb这个数据库,只是还原hellodb中的数据表而已
```

