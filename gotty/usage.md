### gotty 部署
1、下载稳定版 gotty 二进制安装文件
[Gotty - GitHub](https://github.com/yudai/gotty)

2、解压安装包
```angular2html
tar -zxf gotty_2.0.0-alpha.3_linux_amd64.tar.gz
```

3、启动 gotty
pod 启动方式：
```angular2html
nohup ./gotty -w -p 9988 --permit-arguments --reconnect kubectl exec -it &
```
容器启动方式：
```angular2html
nohup ./gotty -w -p 8888 --permit-arguments --reconnect docker exec -it &
```
主机启动方式：
```angular2html
 nohup ./gotty -w -p 8899 --permit-arguments --reconnect sh &
```

4、在浏览器中访问 
命令为：
```angular2html
kubectl exec -it nginx-567d57d4c4-5spp9 -n default bash
```
浏览器中访问 pod 的 url
```angular2html
http://172.20.26.150:8899/?arg=nginx-567d57d4c4-5spp9&arg=-n&arg=default&arg=bash
```
命令为：
```angular2html
docker exec -it b96f8e834f51 bash
```
浏览器中访问 docker 容器 url
```angular2html
http://172.20.26.150:8899/?arg=b96f8e834f51&arg=bash
```

浏览器中访问宿主机的 url
```angular2html
http://172.20.26.150:8899/?
```

需要在每一台机器上都运行一个探针，在一个 gotty 容器中启动三个 gotty 进程，分别在不同的端口监听容器、pod和宿主机的控制台请求。