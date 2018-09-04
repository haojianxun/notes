httpd2.4编译安装

准备工作:

安装Development tools包组和pcre-devel

```
yum -y groupinstall "Development tools"
yum -y install pcre-devel
```

下载要编译的包

apr-1.5.0.tar.bz2,apr-util-1.5.2.tar.bz2,httpd-2.4.9.tar.bz2

解压

```
tar xvf apr-1.5.0.tar.bz2
cd apr-1.5.0
./configure --prefix=/usr/local/apr
tar xf apr-util-1.5.2.tar.bz2 
cd apr-util-1.5.2
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && make install

tar xf httpd-2.4.9.tar.bz2
cd httpd-2.4.9
./configure --prefix=/usr/local/apache --sysconfdir=/etc/httpd24 --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util/ --enable-modules=most --enable-mpms-shared=all --with-event

make && make install

导出头文件
ln -sv /usr/local/apache/include /usr/include/httpd
导出man文档
vim /etc/man.config   加上 MANPATH /usr/local/apache/man
输出二进制程序
vim /etc/profile.d/httpd.sh
export PATH=/usr/local/apache/bin:$PATH
. /etc/profile.d/httpd.sh

ldconfig -p 可以查看当前系统下所有的库文件

```



