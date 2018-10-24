# 如何查看ansible命令执行的详细过程

比如执行的ping命令

```
ansible all -m ping -vvv | grep chmod    //查看有修改权限的操作
ansible all -m ping -vvv|grep rm  //查看有删除的操作
```

