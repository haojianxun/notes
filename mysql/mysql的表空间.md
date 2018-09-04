# mysql的表空间

在`/var/lib/mysql` 目录下可以看到mysql各个数据库,  cd进数据库之后就是数据表,里面放着的是表结构,他们以`.frm` 结尾,真正的数据是不放在这里的,是放到`/var/lib/mysql` 下的`ibdata1` 中的,这是一个表空间