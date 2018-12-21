### govendor 的使用

1、安装 govendor
```angular2html
go get -u -v github.com/kardianos/govendor
```
2、进入项目路径下
```angular2html
wanghuan$ pwd
/mtransformd
wanghuan$ ls
Makefile        conf.prod.yml   src
```
3、初始化 vemndor 目录
```angular2html
govendor init
```
4、添加本工程使用到的依赖包
```angular2html
govendor add +external
或
govendor add +e
```

vendor.json：govendor 配置文件，记录依赖包列表