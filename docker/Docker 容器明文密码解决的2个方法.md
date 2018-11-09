# Docker 容器明文密码解决的2个方法

以下内容来源于网络

## 第一种方法: 替换volume

假设一位用户希望创建一个密码不会泄漏的 MySQL 容器，密码为 daocloud。为了弥合明文密码问题，绕过环境变量，我们可以按照以下三个步骤来完成。

1. 创建两个 MySQL 容器 MySQL1 与 MySQL2，MySQL 的 root 密码分别为 daocloud 与 docker；

2. 待 MySQL1 启动完毕，使用`docker stop` 命令停止 MySQL1 容器，并将 MySQL1 容器的 volume1 全部拷贝出来，最终使用`docker rm` 命令删除 MySQL1 容器；

3. 待 MySQL2 启动完毕，使用`docker stop` 命令停止 MySQL2 容器，并将 MySQL2 容器 volume2 内的文件全部删除，接着将 volume1 的内容拷贝至 volume2 下，最终启动 MySQL2。

通过以上三个步骤，我们直接交付 MySQL2 容器，此时 MySQL2 容器中 MySQL 的 root 密码为 daocloud，即目标达成。虽然 MySQL2 容器的环境变量 MYSQL_ROOT_PASSWORD 依旧是 docker，但是 MySQL 引擎使用的密文密码已经转变为 daocloud，交付完毕的 MySQL2 容器中不存在任何有关字符串 daocloud 的`明文`信息，同时无需再使用的 MySQL1 容器也被我们删除。

上述流程的执行，可以很巧妙的通过`替换volume` 的方式，完成密文的转移，同时使得明文环境变量的失效。



## 第二种方法:secret 

为了解决这个问题，docker swarm 提供了 secret 机制，允许将敏感信息加密后保存到 secret 中，用户可以指定哪些容器可以使用此 secret。

如果使用 secret 启动 MySQL 容器，方法是：

1. 在 swarm manager 中创建 secret `my_secret_data`，将密码保存其中。

```
echo "my-secret-pw" | docker secret create my_secret_data -
```

2. 启动 MySQL service，并指定使用 secret `my_secret_data`。

```
docker service create \
        --name mysql \
        --secret source=my_secret_data,target=mysql_root_password \
        -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
        mysql:latest
```



① source 指定容器使用 secret 后，secret 会被解密并存放到容器的文件系统中，默认位置为 /run/secrets/<secret_name>。`--secret source=my_secret_data,target=mysql_root_password` 的作用就是指定使用 secret `my_secret_data`，然后把器解密后的内容保存到容器 `/run/secrets/mysql_root_password` 文件中，文件名称 `mysql_root_password` 由 `target` 指定。

② 环境变量 `MYSQL_ROOT_PASSWORD_FILE` 指定从 `/run/secrets/mysql_root_password` 中读取并设置 MySQL 的管理员密码。

这里大家可能有这么两个疑问：

1. 问：在第一步创建 secret 时，不也是使用了明文吗？这跟在环境变量中直接指定密码有什么不同呢？

答：在我们的例子中创建 secret 和使用 secret 是分开完成的，其好处是将密码和容器解耦合。secret 可以由专人（比如管理员）创建，而运行容器的用户只需使用 secret 而不需要知道 secret 的内容。也就是说，例子中的这两个步骤可以由不同的人在不同的时间完成。

1. 问：secret 是以文件的形式 mount 到容器中，容器怎么知道去哪里读取 secret 呢？

答：这需要 image 的支持。如果 image 希望它部署出来的容器能够从 secret 中读取数据，那么此 image 就应该提供一种方式，让用户能够指定 secret 的位置。最常用的方法就是通过环境变量，Docker 的很多官方 image 都是采用这种方式。比如 MySQL 镜像同时提供了 `MYSQL_ROOT_PASSWORD` 和 `MYSQL_ROOT_PASSWORD_FILE` 两个环境变量。用户可以用 `MYSQL_ROOT_PASSWORD` 显示地设置管理员密码，也可以通过 `MYSQL_ROOT_PASSWORD_FILE` 指定 secret 路径。