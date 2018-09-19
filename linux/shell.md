### linux 命令
1、杀死某个进程
```angularjs
kill -9 $(ps -ef | grep -v grep | grep nginx | awk '{print $2}')
```

2、
