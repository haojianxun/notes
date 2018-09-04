# python学习笔记

- 查看一个变量的引用计数 

  ```python
  import sys
  sys.getrefcount(VAR)   #VAR指的是你要查询的变量名称
  ```

- 如何查看一个类型有哪些方法呢

  ```python
  dir()   #括号中可以跟你要查看的变量或者类型
  ```

- 如何查看一个方法的使用帮助

  ```python
  help()
  ```

- 浅复制和深复制的区别

  ```
  浅复制是仍然指向原来列表的,原来列表的元素发生改变了,浅复制也要改变
  深复制是递归复制一个新的,深复制可使用copy模块中的deepcopy()实现
  ```

- 不想换行打印,可以在print后面加个逗号

  ```python
  比如:
  x = 0 , y = 100
  while x < y:
      print x,
      x += 1
  ```

  ​