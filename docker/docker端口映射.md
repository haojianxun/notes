#docker端口映射

docker0是NAT网桥 , 因此获得是私有网络地址

可以把容器想象成宿主机NAT服务背后的主机

如果开放容器给外部网络访问的话 , 需要在宿主机上为其指定DNAT规则 , 比如:

- 对宿主机某ip地址的访问全部映射给某容器

  ```
  -A PREROUTING -d 主机IP -j DNAT --to-destination 容器IP
  ```

- 对宿主机某IP地址的某端口的访问映射给某容器地址的某端口

  ```
  -A PREROUTING -d 主机IP -p {tcp|udp} --dport 主机端口 -j DNAT --to-desitination 容器IP:容器端口
  ```

`docker run`命令使用`-p`选项即可实现端口映射 无需手动添加规则

option

| -p                                 | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| -p <containerPort>                 | 将指定的容器端口映射到主机 所有 地址的一个动态端口           |
| -p <hostPort>:<containerPort>      | 将容器端口<containerPort>映射为指定主机端口<hostPort>        |
| -p <ip>::<containerPort>           | 将容器的端口<containerPort>映射为主机指定<IP>的动态端口 , 其中2个冒号表示: 2个冒号中间代表主机端口 ,这里为空,代表随机 |
| -p <ip>:<hostPort>:<containerPort> | 将容器端口<containerPort>映射为主机指定<ip>的指定端口<hostPort> |

*"动态端口"指的是随机端口 , 具体可以用`docker port CONTAINER_NAME`命令查看*

