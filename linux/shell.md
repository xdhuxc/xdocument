### linux 命令
1、杀死某个进程
```angularjs
kill -9 $(ps -ef | grep -v grep | grep nginx | awk '{print $2}')
```

2、查找占用某端口的进程
```angular2html
netstat -tlnup|grep 35888|awk -F ' ' '{print $7}'|cut -d / -f2
```
