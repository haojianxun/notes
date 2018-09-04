# python技巧

- python的开发环境要迁移的时候可以用

  ```
  pip freeze > requirements.txt #把当前环境的包列出来导入到requirements.txt中
  pip install -r requirements.txt  #从requirements.txt中读取包并且安装
  ```

  ​