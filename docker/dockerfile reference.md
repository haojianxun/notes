# dockerfile reference

[TOC]



## 用法

`docker build` 这个命令可以通过dockerfile和context来构建一个镜像,context是通过`PATH` 和`URL` 来设置的,PATH是一个在本地文件系统的目录,URL是一个本地git仓库

context是递归处理的,所以一个`PATH`是包括其子目录的,一个`URL`是包括其子仓库的,下面的一个例子展示了用当前目录作为context来运行build命令的效果

```
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```

通常最好的办法是找一个空目录作为context, 并且把dockerfile扔进这个空目录,里面仅仅增加构建dockerfile所需的文件

> **Warining**: 不要用root目录来作为`PATH` , 因为他会把整个root目录 "/" 的内容仍给Docker daemon进程

通常dockerfile位于context目录下,可以使用`-f` 来指定具体的dockerfile路径,但是默认的一般是当前目录的dockerfile ,可以不用写 比如:

```
docker build -f /path/to/a/Dockerfile .  
```

也可以指定仓库和tag来保存新构建的镜像

```
docker build -t shykes/myapp .
```

也可以使用`-t`来保存到多个仓库中

```
docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```

## Format

指令是不区分大小写的, 但是都约定成俗的都是大写

每个dockerfile开始的指令都是以`FROM` 开始

### Environment replacement

环境变量由`ENV` 语句设置, 可以用`$variable_name` or `${variable_name}`设置

- `${variable:-word}` indicates that if `variable` is set then the result will be that value. If `variable` is not set then `word` will be the result.  //  如果设置了变量,就用变量的,如果没有设置,就用现在的设置的,相当于设置空白默认值
- `${variable:+word}` indicates that if `variable` is set then `word` will be the result, otherwise the result is the empty string. //如果设置了,就用设置的,没有设置,就是空的

例子

```
FROM busybox
ENV foo /bar  # 设置变量 foo = /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux      #加上\ 就会使变量变成字面意思
```







