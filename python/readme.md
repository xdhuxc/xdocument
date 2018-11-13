### 源代码安装
1、下载安装包
```angularjs

```
2、编译安装
```angularjs
./configure --prefix=/usr/local/python37
make -j 4
make install
```
执行 make install 时报错如下：
```angularjs
zipimport.ZipImportError: can't decompress data; zlib not available
```
在CentOS以及其他的Linux系统中遇到安装包安装错误的原因，大多数都是因为缺少依赖包导致的，所以对于错误：zipimport.ZipImportError: can’t decompress data，是因为缺少zlib 的相关工具包导致的，因此，执行如下命令安装 zlib 相关工具包：
```angularjs
yum -y install zlib*
```

报错如下：
```angularjs
Traceback (most recent call last):
  File "/root/Python-3.7.1/Lib/runpy.py", line 193, in _run_module_as_main
    "__main__", mod_spec)
  File "/root/Python-3.7.1/Lib/runpy.py", line 85, in _run_code
    exec(code, run_globals)
  File "/root/Python-3.7.1/Lib/ensurepip/__main__.py", line 5, in <module>
    sys.exit(ensurepip._main())
  File "/root/Python-3.7.1/Lib/ensurepip/__init__.py", line 204, in _main
    default_pip=args.default_pip,
  File "/root/Python-3.7.1/Lib/ensurepip/__init__.py", line 117, in _bootstrap
    return _run_pip(args + [p[0] for p in _PROJECTS], additional_paths)
  File "/root/Python-3.7.1/Lib/ensurepip/__init__.py", line 27, in _run_pip
    import pip._internal
  File "/tmp/tmpye2i7k6a/pip-10.0.1-py2.py3-none-any.whl/pip/_internal/__init__.py", line 42, in <module>
  File "/tmp/tmpye2i7k6a/pip-10.0.1-py2.py3-none-any.whl/pip/_internal/cmdoptions.py", line 16, in <module>
  File "/tmp/tmpye2i7k6a/pip-10.0.1-py2.py3-none-any.whl/pip/_internal/index.py", line 25, in <module>
  File "/tmp/tmpye2i7k6a/pip-10.0.1-py2.py3-none-any.whl/pip/_internal/download.py", line 39, in <module>
  File "/tmp/tmpye2i7k6a/pip-10.0.1-py2.py3-none-any.whl/pip/_internal/utils/glibc.py", line 3, in <module>
  File "/root/Python-3.7.1/Lib/ctypes/__init__.py", line 7, in <module>
    from _ctypes import Union, Structure, Array
ModuleNotFoundError: No module named '_ctypes'
make: *** [install] Error 1
```
3.7版本需要一个新的包libffi-devel，安装此包之后再次进行编译安装即可
执行如下命令：
```angularjs
yum install -y libffi-devel
```

3、设置 python 3.7 为默认的 python 解释器
删除原来的符号链接
```angularjs
rm -f /usr/bin/python
```
设置新的 python 链接。
```angularjs
ln -s -T /usr/local/python37/bin/python3.7 /usr/bin/python
```

### virtualenvwrapper 

#### windows 系统下

1、安装 virtualenvwrapper
```angularjs
pip install virtualenvwrapper
```
或
```angularjs
pip install virtualenvwrapper-win
```
2、virtualenvwrapper 的使用

命令：
workon：列出所有的虚拟环境
workon xflask：进入虚拟环境 xflask

mkvirtualenv xflask：创建虚拟环境 xflask，统一存放在 Envs 目录下，如果没有虚拟环境目录，则创建 ~/Envs 目录，在其下面创建虚拟环境目录

mkvirtualenv --python=C:\xdhuxc\Python37-32\python.exe xflask

deactivate：退出虚拟环境。

rmvirtualenv：删除虚拟环境。

lsvirtualenv：列出所有的虚拟环境，和不带参数的 workon 效果一样。

配置：
修改 Envs 目录为指定目录，添加如下环境变量：
```angularjs
WORKON_HOME=E:\PycharmProject\Envs
```

### Python 开源项目地址

https://www.cnblogs.com/botoo/p/8294848.html

https://www.lfd.uci.edu/~gohlke/pythonlibs/#


