## 在python中 如何在一个列表中选择某些特定元素组成一个新列表

比如

```python
import os
filelist = os.listdir('/var/log')
filelist2 = [i for i in filelist if i.endswith('.log')]
```

