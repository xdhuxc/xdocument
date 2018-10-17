### Python 开源项目地址

https://www.cnblogs.com/botoo/p/8294848.html


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


