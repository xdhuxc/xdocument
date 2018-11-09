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

