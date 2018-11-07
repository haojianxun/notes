# docker是如何做到资源限制的

**使用的便是Linux Cgroups技术   全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等**



在 Linux 中，Cgroups 给用户暴露出来的操作接口是文件系统，即它以文件和目录的方式组织在操作系统的 `/sys/fs/cgroup` 路径下

```
[root@aliyun-node01 ~]# mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)

```



cgroups限制的种类有很多, 具体我们可以看到在`/sys/fs/cgroup`下有许多子目录 ,  进入这些子目录会看到许多文件 , 这些都是被限制的具体方法



进入这些子目录下 , 如果创建一个文件夹的话, 会发现这个新创建的文件夹下面有许多文件 , 几乎和子目录下的文件一样 



那系统是如何做到限制应用进程的资源配置的呢



以限制进程cpu为例

写一个死循环

```bash
while : ; do : ; done &
[1] 31472
```

这样的运行的话 cpu就会被吃满  , 可以使用`top`命令查看

我们进入cgroups的cpu子目录下, 创建一个文件夹

```
cd /sys/fs/cgroup/cpu

mkdir test
```



进入这个文件夹

```bash
[root@aliyun-node01 cpu]# cd test/
[root@aliyun-node01 test]# ls
cgroup.clone_children  cgroup.procs  cpuacct.usage         cpu.cfs_period_us  cpu.rt_period_us   cpu.shares  notify_on_release
cgroup.event_control   cpuacct.stat  cpuacct.usage_percpu  cpu.cfs_quota_us   cpu.rt_runtime_us  cpu.stat    tasks
```



查看这个文件夹的文件 , 看控制组的CPU quota

```bash
cat cpu.cfs_quota_us   //看到的结果是-1 , 表明没有任何限制
cat cpu.cfs_period_us  //看到CPU period 是默认的 100 ms（100000 us）
```



比如，向 test控制组里的 cfs_quota 文件写入 20 ms（20000 us）：

```bash
echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
```

它意味着在每 100 ms 的时间里，被该控制组限制的进程只能使用 20 ms 的 CPU 时间，也就是说这个进程只能使用到 20% 的 CPU 带宽



接下来，我们把被限制的进程的 PID 写入 控制组里的 tasks 文件，上面的设置就会对该进程生效了：

```bash
$ echo 31472 > /sys/fs/cgroup/cpu/container/tasks 
```

我们可以用 top 指令查看一下：

```
$ top
%Cpu0 : 20.3 us, 0.0 sy, 0.0 ni, 79.7 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

可以看到cpu使用率降到了20%



docker的资源限制也是这样做的  , 它在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了

至于docker要控制什么 , 就靠`docker run`的时候加上参数了

比如:

```
docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
```

在启动这个容器后，我们可以通过查看 Cgroups 文件系统下，CPU 子系统中，“docker”这个控制组里的资源限制文件的内容来确认：

```bash
cat /sys/fs/cgroup/cpu/docker/a2533b3eed3cf30debc849ca30277f728955675bac267e271268edebc8b12b9e/cpu.cfs_period_us 
100000


cat /sys/fs/cgroup/cpu/docker/a2533b3eed3cf30debc849ca30277f728955675bac267e271268edebc8b12b9e/cpu.cfs_quota_us 
20000
```

其中`a2533b3eed3cf30debc849ca30277f728955675bac267e271268edebc8b12b9e`是这个启动docker容器的CONTAINER ID



以下为在我机器上的具体表现

```bash
[root@aliyun-node01 docker]# pwd
/sys/fs/cgroup/cpu/docker



[root@aliyun-node01 docker]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                            NAMES
da27b7caa4bd        yinaoxiong/oneindex   "docker-php-entrypoi…"   2 weeks ago         Up 2 weeks          9000/tcp, 0.0.0.0:8888->80/tcp   onedrive02
a2533b3eed3c        yinaoxiong/oneindex   "docker-php-entrypoi…"   4 weeks ago         Up 4 weeks          9000/tcp, 0.0.0.0:8080->80/tcp   onedrive


[root@aliyun-node01 docker]# ls
a2533b3eed3cf30debc849ca30277f728955675bac267e271268edebc8b12b9e  cpuacct.usage_percpu  cpu.stat
cgroup.clone_children                                             cpu.cfs_period_us     da27b7caa4bdaa1adaf0a02642a4e6f830498687f9b2c44c3cfcb2c6e7bdce99
cgroup.event_control                                              cpu.cfs_quota_us      notify_on_release
cgroup.procs                                                      cpu.rt_period_us      tasks
cpuacct.stat                                                      cpu.rt_runtime_us
cpuacct.usage                                                     cpu.shares

```

