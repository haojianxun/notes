## [PY]如何把一个目录下的所有文件名写入到另一个新建文件中去

```python
import os
l1 = [i+'\n' for i in os.listdir('/etc/')]
l2.writelines(l1)
l2.flush()
```

