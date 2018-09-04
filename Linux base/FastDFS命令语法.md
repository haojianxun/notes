# FastDFS命令语法

FastDFS的语法可以直接看,在命令行中直接敲fdfs然后补2下tab键就可以看到许多fdfs的命令了,获取帮助的办法就是命令之后加-h

## `fdfs_test` 

```
fdfs_test -h  #测试用的

Usage: fdfs_test <config_file> <operation>
	operation: upload, download, getmeta, setmeta, delete and query_servers
	
例如:
fdfs_test /etc/fdfs/client.conf upload /etc/fstab  #上传fstab

```

## `fdfs_upload_file` 

```
fdfs_upload_file

Usage: 
fdfs_upload_file <config_file> <local_filename> [storage_ip:port] [store_path_index]
```



## `fdfs_appender_test` 

```
Usage: fdfs_appender_test <config_file> <local_filename> [FILE | BUFF | CALLBACK]
```

## `fdfs_delete_file` 

```
Usage: fdfs_delete_file <config_file> <file_id>
```

## `fdfs_download_file` 

```
Usage: 
fdfs_download_file <config_file> <file_id> [local_filename] [<download_offset> <download_bytes>]
```

.....

其余的可以在命令行看