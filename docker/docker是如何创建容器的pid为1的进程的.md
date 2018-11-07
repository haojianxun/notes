# docker是如何创建容器的pid为1的进程的

举个例子来说明:

```
docker run -it busybox /bin/sh
```

此命令会进入busybody的交互式命令行



我们在运行上面命令的时候 , 对于操作系统来讲是个进程 , 操作系统会分配一个pid号给这个进程 ,比如这个进程号是PID=111 , 对于操作系统全局来讲它是pid=111的一个进程 , 但是我们进入容器执行命令`ps`  , 会发现`/bin/sh`的pid竟然等于1 

这种就是docker的namespace机制 , 对于全局来讲 , 这条docker命令的的pid可能是111 , 但是对于容器内部来讲 , 它营造一个虚假的空间,  使得`/bin/sh` 的pid进程等于1



其中用到的技术就是Linux的创建线程的系统调 , 函数是 clone()

比如：

```c
int pid = clone(main_function, stack_size, SIGCHLD, NULL); 
```

这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。

而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如：

```c
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```

这时，新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 111



 " 容器进程 "，是 Docker 创建的一个容器初始化进程 (dockerinit)，而不是应用进程 (ENTRYPOINT + CMD)。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程。



容器 只是一个特殊进程而已