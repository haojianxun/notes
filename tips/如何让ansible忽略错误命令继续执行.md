# 如何让ansible忽略错误命令继续执行

有2个种方法:

1. ```
   tasks:
     - name: run this command and ignore the result
       shell:/usr/bin/command ||/bin/true
   ```

2. 使用ignore_errors来忽略错误信息

   ```
   tasks:
     - name: run this command and ignore the result
       shell: /usr/bin/command
       ignore_errors: True
   ```
