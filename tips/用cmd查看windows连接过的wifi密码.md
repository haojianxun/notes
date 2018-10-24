# 用cmd查看windows连接过的wifi密码

运行cmd , 输入以下代码

```
for /f "skip=9 tokens=1,2 delims=:" %i in ('netsh wlan show profiles') do  @echo %j | findstr -i -v echo | netsh wlan show profiles %j key=clear
```

