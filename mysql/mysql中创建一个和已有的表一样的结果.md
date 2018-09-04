# mysql中创建一个和已有的表一样的结果

```
CREATE TABLE test LIKE mytable;


CREATE TABLE test1 SELECT * FROM mytable;
```

