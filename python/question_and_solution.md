#### 1、以下代码中，先重新加载sys模块，再设置编码，而不是先设置编码，再加载sys模块的原因：
```angularjs
import sys
reload(sys)
sys.setdefaultencoding('utf8')
```

因为 python 运行的时候首先加载了 Lib/site.py，而在 site.py 文件中有如下代码：
```angularjs
if hasattr(sys, "setdefaultencoding"):
    del sys.setdefaultencoding
```
在sys模块加载后，setdefaultencoding方法被删除了，所以我们需要通过重新导入sys模块来设置系统编码。


####