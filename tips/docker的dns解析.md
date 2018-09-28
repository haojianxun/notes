# docker的dns解析

docker的dns解析一般默认是docker0网桥的地址 这样做的目的是方便在同一主机内的各个容器互相通信  当然也可以自己指定dns地址  在启动docker的时候使用短选项`--dns` 即可指定   进入容器内部`cat /etc/resolv.conf`查看的时候 就会发现默认的已经改变