# dockerfile常用指令

官方文档:https://docs.docker.com/engine/reference/builder/

ENV

`${VAR:-STR}`  :表示var这个变量 如果有值的话 , 就用自己的值 没有的话 就用STR这个值 , 相当于变量原来有值就用原来的 , .没有的话就用我们给出的值STR------ 相当于给变量设置默认值

`${VAR:+STR} `  如果VAR有值 就返回STR



## FROM

```
FROM <repository>[:<tag>]
```

## MAINTAINER

作者信息

## COPY

```
COPY <src> ...<dest> 或
COPY ["<src>",..."<dest>"]
```

- `des`t建议使用绝对路径

- `src`必须是build上下文中的的路径 , 不能是其父目录中的路径

- 如果`src`是目录,其内部文件或子文件或被递归复制 , 但是`src`目录本身不会被复制

- 如果`src`有多个  则`dest`必须以` /` 结尾

- 如果dest不存在 则会被创建 包括父目录路径

## ADD

```
ADD <src> ...<dest> 或
ADD ["<src>",..."<dest>"]
```

- 如果`src`是URL而且`dest`不以`/`结尾 ,则下载的文件直接保存为`dest`  如果`dest`以` /` 结尾 则保存的文件是`dest/filname`
- 如果`src`是本地是一个tar文件 则会被展开为一个目录 ,相当于解压到了`dest`下 如果是url的话 就不会被展开
- `src`有多个 ,则`dest`必须是以`/`结尾 , 如果不是以/结尾 则`src`的内容就会被写入到`dest`

## WORKDIR

用于为`RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`指定工作目录

## EXPOSE

```
EXPOSE <port> [<port>/<protocol>...]
```

暴露的端口只是待暴露 , 没有主动暴露 要想主动暴露被外部访问 可以在`docker run`的时候加上`-P`  如果在docker run的时候用-p的话(这个是小p)  , 可以被覆盖

## ENV

```
ENV <key> <value>
ENV <key>=<value> ...
```

在使用第二种格式的时候 可以使用 `\` 来另起一行作为分割 也可以用来转义空格字符等 ,第二种支持多个变量设置 第一种只能设置一个变量

## RUN

RUN has 2 forms:

- `RUN <command>` (*shell* form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows)

  此进行在容器中PID不为1 , 不能接受unix信号 , 当使用docker stop命令停止容器的时候 他是接受不到SIGTREM信号的

- `RUN ["executable", "param1", "param2"]` (*exec* form)

  不会以`/bin/sh -c` 发起, 所以shell中常用的变量替换,通配符都不支持 ,如果要运行有shell特性命令的话可以使用 `RUN ["/bin/sh","-c","executable", "param1", "param2"]`

一般情况是在docker build 的时候运行

## CMD

The `CMD` instruction has three forms:

- `CMD ["executable","param1","param2"]` (*exec* form, this is the preferred form) 启动的是PID为1的进程 , 可以接受UNIX信号 , 可以接受docker stop发来的信号
- `CMD ["param1","param2"]` (as *default parameters to ENTRYPOINT*)
- `CMD command param1 param2` (*shell* form)

There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD` then only the last `CMD` will take effect.

run命令运行在容器构建过程中 , 而cmd命令是基于dockerfile构建出的镜像运行的命令

cmd命令在于为容器启动默认的运行程序 , 而且运行结束 容器终止, 可以被docker run命令所覆盖

cmd的指令在于为启动的容器运行一个默认的程序 , 而且运行结束后, 容器也终止 , cmd命令可以被docker run的命令行选项所覆盖

只有最后一个cmd还会有效 之前的会被覆盖无效

## ENTRYPOINT

ENTRYPOINT has two forms:

- `ENTRYPOINT ["executable", "param1", "param2"]` (*exec* form, preferred)
- `ENTRYPOINT command param1 param2` (*shell* form)

entrypoint用于为容器指定默认运行程序, 从而使得容器像是一个单独的程序

entrypoint默认用的是/bin/sh -c来启动的  第二种方法后面跟的命令是用shell起的 但是在容器的表现是 后面起的命令pid是1  可以配合cmd命令来使用

docker  run后面跟的参数会附加在entrypoint之后运行  会覆盖cmd的命令  cmd的命令是默认的参数 没有就用他 命令行有参数就不用他

第一种方式 可以 修改默认的启动程序 /bin/sh -c

## HEALTHCHECK

The `HEALTHCHECK` instruction has two forms:

- `HEALTHCHECK [OPTIONS] CMD command` (check container health by running a command inside the container)
- `HEALTHCHECK NONE` (disable any healthcheck inherited from the base image)

The options that can appear before `CMD` are:

- `--interval=DURATION` (default: `30s`)
- `--timeout=DURATION` (default: `30s`)
- `--start-period=DURATION` (default: `0s`)
- `--retries=N` (default: `3`)

## ONBUILD

```
ONBUILD [INSTRUCTION]
```

写在dockerfile文件中 自己build 的时候不会执行  当别人用我们镜像的时候 会触发这个操作