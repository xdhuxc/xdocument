### virtualenv
1、初始化虚拟环境时指定 python 版本，使用 -p 参数指定 python 解释器路径
```angularjs
virtualenv -p C:\xdhuxc\Python37-32\python.exe venv 
```

### 性能分析

https://segmentfault.com/a/1190000007518598 

#### cProfile
```angularjs
python -m cProfile abc.py 
```

#### line_profile
在待检测的函数上面添加 @profile 装饰器
```angularjs
@profile   # 不用管此处的提示 "unresolved reference 'profile'"
def func1():
    sum = 0
    for i in range(100000):
        sum += i
```
运行该脚本
```angularjs
kernprof -l -v xdhuxc.py
```

### 模块安装



