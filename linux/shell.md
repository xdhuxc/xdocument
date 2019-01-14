### linux 命令
1、杀死某个进程
```angularjs
kill -9 $(ps -ef | grep -v grep | grep nginx | awk '{print $2}')
```

2、查找占用某端口的进程
```angular2html
netstat -tlnup|grep 35888|awk -F ' ' '{print $7}'|cut -d / -f2
```

3、置空文件
```angularjs
echo "" > abc.txt # abc.txt 文件内部会有一个空行
或
: > abc.txt # abc.txt 文件内部没有空行 
```

4、切换到 root 用户
```angularjs
sudo su - # 由普通用户无密码切换到 root 用户 
```

5、查看进程及其子进程
```angular2html
# -l：不截断输出，显示整个进程或线程的启动命令
# 3376：要显示的进程 ID
# -a：显示命令行参数
pstree -a -l 3376
```

