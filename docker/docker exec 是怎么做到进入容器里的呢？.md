# docker exec 是怎么做到进入容器里的呢？

先查看当前docker容器进程号

```
docker inspect --format '{{ .State.Pid }}' CONTAINER ID //CONTAINER ID可以用命令docker ps 查看
```

获得pid进程号为

```bash
# docker inspect --format '{{ .State.Pid }}' da27b7caa4bd
10822
```

用这个查看10822进程对于的所有namespace文件

```bash
# ll /proc/10822/ns
lrwxrwxrwx 1 root root 0 Nov  8 17:56 ipc -> ipc:[4026532224]
lrwxrwxrwx 1 root root 0 Nov  8 17:56 mnt -> mnt:[4026532222]
lrwxrwxrwx 1 root root 0 Nov  8 17:56 net -> net:[4026532227]
lrwxrwxrwx 1 root root 0 Nov  8 17:56 pid -> pid:[4026532225]
lrwxrwxrwx 1 root root 0 Nov  8 17:56 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Nov  8 17:56 uts -> uts:[4026532223]
```

可以看到，一个进程的每种 Linux Namespace，都在它对应的 /proc/[进程号]/ns 下有一个对应的虚拟文件，并且链接到一个真实的 Namespace 文件上。

有了这样一个可以“hold 住”所有 Linux Namespace 的文件，我们就可以对 Namespace 做一些很有意义事情了，比如：加入到一个已经存在的 Namespace 当中。

**这也就意味着：一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的，这正是 docker exec 的实现原理。**

而这个操作所依赖的，乃是一个名叫 setns() 的 Linux 系统调用

简单是实现是通过 open() 系统调用打开了指定的 Namespace 文件，并把这个文件的描述符 fd 交给 setns() 使用。在 setns() 执行后，当前进程就加入了这个文件对应的 Linux Namespace 当中了



比如 Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net