# FastDFS文件同步机制

在FastDFS的服务器端配置文件中，bind_addr这个参数用于需要绑定本机IP地址的场合。只有这个参数和主机特征相关，其余参数都是可以统一配置的。在不需要绑定本机的情况下，为了便于管理和维护，建议所有tracker server的配置文件相同，同组内的所有storage server的配置文件相同。

tracker server的配置文件中没有出现storage server，而storage server的配置文件中会列举出所有的tracker server。这就决定了storage server和tracker server之间的连接由storage server主动发起，storage server为每个tracker server启动一个线程进行连接和通讯，这部分的通信协议请参阅《FastDFS HOWTO -- Protocol》中的“2. storage server to tracker server command”。

tracker server会在内存中保存storage分组及各个组下的storage server，并将连接过自己的storage server及其分组保存到文件中，以便下次重启服务时能直接从本地磁盘中获得storage相关信息。storage server会在内存中记录本组的所有服务器，并将服务器信息记录到文件中。tracker server和storage server之间相互同步storage server列表：
  1. 如果一个组内增加了新的storage server或者storage server的状态发生了改变，tracker server都会将storage server列表同步给该组内的所有storage server。以新增storage server为例，因为新加入的storage server主动连接tracker server，tracker server发现有新的storage server加入，就会将该组内所有的storage server返回给新加入的storage server，并重新将该组的storage server列表返回给该组内的其他storage server；
  2. 如果新增加一台tracker server，storage server连接该tracker server，发现该tracker server返回的本组storage server列表比本机记录的要少，就会将该tracker server上没有的storage server同步给该tracker server。

同一组内的storage server之间是对等的，文件上传、删除等操作可以在任意一台storage server上进行。文件同步只在同组内的storage server之间进行，采用push方式，即源服务器同步给目标服务器。以文件上传为例，假设一个组内有3台storage server A、B和C，文件F上传到服务器B，由B将文件F同步到其余的两台服务器A和C。我们不妨把文件F上传到服务器B的操作为源头操作，在服务器B上的F文件为源头数据；文件F被同步到服务器A和C的操作为备份操作，在A和C上的F文件为备份数据。同步规则总结如下：
  1. 只在本组内的storage server之间进行同步；
  2. 源头数据才需要同步，备份数据不需要再次同步，否则就构成环路了；
  3. 上述第二条规则有个例外，就是新增加一台storage server时，由已有的一台storage server将已有的所有数据（包括源头数据和备份数据）同步给该新增服务器。

storage server有7个状态，如下：
   FDFS_STORAGE_STATUS_INIT      :初始化，尚未得到同步已有数据的源服务器
   FDFS_STORAGE_STATUS_WAIT_SYNC :等待同步，已得到同步已有数据的源服务器
   FDFS_STORAGE_STATUS_SYNCING   :同步中
   FDFS_STORAGE_STATUS_DELETED   :已删除，该服务器从本组中摘除（注：本状态的功能尚未实现）
   FDFS_STORAGE_STATUS_OFFLINE   :离线
   FDFS_STORAGE_STATUS_ONLINE    :在线，尚不能提供服务
   FDFS_STORAGE_STATUS_ACTIVE    :在线，可以提供服务

当storage server的状态为FDFS_STORAGE_STATUS_ONLINE时，当该storage server向tracker server发起一次heart beat时，tracker server将其状态更改为FDFS_STORAGE_STATUS_ACTIVE。

组内新增加一台storage server A时，由系统自动完成已有数据同步，处理逻辑如下：
  1. storage server A连接tracker server，tracker server将storage server A的状态设置为FDFS_STORAGE_STATUS_INIT。storage server A询问追加同步的源服务器和追加同步截至时间点，如果该组内只有storage server A或该组内已成功上传的文件数为0，则没有数据需要同步，storage server A就可以提供在线服务，此时tracker将其状态设置为FDFS_STORAGE_STATUS_ONLINE，否则tracker server将其状态设置为FDFS_STORAGE_STATUS_WAIT_SYNC，进入第二步的处理；
  2. 假设tracker server分配向storage server A同步已有数据的源storage server为B。同组的storage server和tracker server通讯得知新增了storage server A，将启动同步线程，并向tracker server询问向storage server A追加同步的源服务器和截至时间点。storage server B将把截至时间点之前的所有数据同步给storage server A；而其余的storage server从截至时间点之后进行正常同步，只把源头数据同步给storage server A。到了截至时间点之后，storage server B对storage server A的同步将由追加同步切换为正常同步，只同步源头数据；
  3. storage server B向storage server A同步完所有数据，暂时没有数据要同步时，storage server B请求tracker server将storage server A的状态设置为FDFS_STORAGE_STATUS_ONLINE；
     4 当storage server A向tracker server发起heart beat时，tracker server将其状态更改为FDFS_STORAGE_STATUS_ACTIVE。