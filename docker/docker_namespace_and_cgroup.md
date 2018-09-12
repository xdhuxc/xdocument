### Linux namespace 概念
Linux 内核从版本 2.4.19 开始陆续引入了 namespace 的概念，其目的是将某个特定的全局系统资源，通过抽象的方式，使得 namespace 中的进程看起来拥有它们自己的隔离的全局系统资源实例。

Linux 内核中实现了六种 namespace，按照引入的先后顺序，列表如下：
命名空间名称 | 引入的相关内核版本 | 被隔离的全局系统资源 | 在容器语境下的隔离效果
---|---|---|---|---
Mount 命名空间 | Linux 2.4.19 | 文件系统挂载点 | 每个容器能看到不同的文件系统层次结构
UTS 命名空间 | Linux 2.6.19 | hostname 和 domainname | 每个容器可以有自己的 hostname 和 domainname
IPC 命名空间 | Linux 2.6.19 | 特定的进程间通信资源，包括 System V IPC 和 POSIX message queues | 每个容器有其自己的 System V IPC 和 POSIX 消息队列文件系统。因此，只有在同一个 IPC namespace 中的进程之间才能相互通信。
PID 命名空间 | Linux 2.6.24 | 进程 ID 数字空间（process ID number space）| 每个 PID namespace 中的进程可以有其独立的 PID；每个容器可以有其 PID 为 1 的 root 进程，也使得容器可以在不同的主机之间迁移，因为 namespace 中的进程 ID 与主机无关了，这也使得容器中的每个进程有两个 PID：容器中的 PID 和主机上的 PID。 
Network 命名空间| 始于 Linux 2.6.24，完成于 Linux 2.6.29 | 网络相关的系统资源 | 每个容器拥有其独立的网络设备、IP地址、IP路由表、/proc/net 目录、端口号等等。这也使得一个主机上多个容器内的同一个应用都可以绑定到各自容器的 80 端口上。
User 命名空间 | 始于 Linux 2.6.23，完成于 Linux 3.8 | 用户和组 ID 空间 | 在 user namespace 中的进程的用户和组 ID 可以和在宿主机上不同；每个容器可以有不同的用户和组ID，一个主机上的非特权用户可以成为 user namespace 中的特权用户。

简单来讲，处于某个命名空间中的进程，能看到独立的它自己的隔离的某些特定系统资源。

### docker 容器使用 linux namespace 做运行环境隔离
当 docker 创建一个容器时，它会创建新的以上六种 namespace 的实例，然后把容器中的所有进程放到这些 namespace 之中，使得 docker 容器中的进程只能看到隔离的系统资源。

```
[root@xdhuxc ns]# pwd
/proc/3300/ns          # 3300 为容器在宿主机上的进程ID
[root@xdhuxc ns]# ll
total 0
lrwxrwxrwx 1 1000 1000 0 Jul 11 08:47 ipc -> ipc:[4026532175]  # 1000 为容器在宿主机上的用户
lrwxrwxrwx 1 1000 1000 0 Jul 11 08:47 mnt -> mnt:[4026532173]
lrwxrwxrwx 1 1000 1000 0 Jul 11 08:37 net -> net:[4026532178]
lrwxrwxrwx 1 1000 1000 0 Jul 11 08:47 pid -> pid:[4026532176]
lrwxrwxrwx 1 1000 1000 0 Jul 11 09:00 user -> user:[4026531837]
lrwxrwxrwx 1 1000 1000 0 Jul 11 08:47 uts -> uts:[4026532174]
```

#### PID 命名空间
创建容器 zookeeper 
```
docker run -d \
    --privileged=true \
    --restart=always \
    --name zookeeper \
    zookeeper:latest
```
在容器内的 PID 是 1，PPID 为 0。
```
[root@xdhuxc ~]# docker exec -it zookeeper ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
zookeep+     1     0  0 14:53 ?        00:00:01 /usr/lib/jvm/java-1.8-openjdk/jr
root        47     0  0 14:57 pts/0    00:00:00 ps -ef
```
在容器外的 PID 是 3391，PPID 是 3363，即 docker-containerd-shim 进程。
```
[root@xdhuxc ~]# ps -ef | grep -v grep | grep zookeeper
1000      3391  3363  0 22:53 ?        00:00:02 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
```
3363 是 docker-containerd-shim 进程的 PID。
```
[root@xdhuxc ~]# ps -ef | grep -v grep | grep 3363
root      3363 21666  0 22:53 ?        00:00:00 docker-containerd-shim -namespace moby -workdir /var/lib/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/81cae3d22fd7ce29c2b49292bce16ac52f0cca8ea26b21b8f34e551deb7faef2 -address /var/run/docker/containerd/docker-containerd.sock -containerd-binary /usr/bin/docker-containerd -runtime-root /var/run/docker/runtime-runc -systemd-cgroup
1000      3391  3363  0 22:53 ?        00:00:02 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
```

我们能看到，同一个进程，在容器内外的 PID 是不同的。
* 在容器内 PID 是 1，PPID 是 0.
* 在容器外 PID 是 3391，PPID 是 3363，即 docker-containerd-shim 进程

关于 containerd，containerd-shim 和 container 的关系：

缺图

* docker 引擎管理着镜像，然后移交给 containerd 运行，containerd 再使用 runC 运行容器。
* containerd 是一个简单的守护进程，它可以使用 runC 管理容器，使用 gRPC 暴露容器的其他功能。它管理容器的开始、停止、暂停和销毁。由于容器运行时是孤立的引擎，引擎最终能够启动和升级而无需重新启动容器。
* runC 是一个轻量级的工具，它是用来运行容器的，只用来做这一件事，并且这一件事要做好。runC 基本上是一个小命令行工具，且它可以不用通过 docker 引擎，直接就可以使用容器。

因此，容器中的主应用在宿主机上的父进程是 containerd-shim，是它通过工具 runC 来启动这些进程的。

这也能看出来，pid 命名空间通过将宿主机上 PID 映射为容器内的 PID，使得容器内的进程看起来有个独立的 PID 空间。

#### UTS 命名空间
容器可以有自己的 hostname 和 domainname
```
[root@xdhuxc ~]# hostname
xdhuxc             # 宿主机的主机名
[root@xdhuxc ~]# docker exec -it zookeeper hostname
81cae3d22fd7       # zookeeper 容器的主机名
```

#### user 命名空间
在 docker 1.10 版本之前，是不支持 user 命名空间的，也就是说，默认地，容器内的进程的运行用户就是宿主机上的 root 用户。这样的话，当宿主机上的文件或者目录作为挂载卷被映射到容器以后，容器内的进程其实是有 root 的几乎所有权限去修改这些宿主机上的目录的，这会有很大的安全问题。
```
[root@xdhuxc ~]# id
uid=0(root) gid=0(root) groups=0(root)
[root@xdhuxc ~]# docker exec -it zookeeper id
uid=0(root) gid=0(root)
```
此时以如下命令启动一个容器
```
docker run -d -v /bin:/host/bin --name zookeeper zookeeper:latest
```

默认情况下，是这样子的：
```
[root@xdhuxc ~]# ps -ef | grep zookeeper
1000      3300  3284  0 08:37 ?        00:00:02 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
root      4323  1602  0 08:48 pts/1    00:00:00 grep --color=auto zookeeper
[root@xdhuxc ~]# docker exec -it zookeeper ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
zookeep+     1     0  0 00:37 ?        00:00:02 /usr/lib/jvm/java-1.8-openjdk/jr
root        43     0  0 00:48 pts/0    00:00:00 ps -ef
[root@xdhuxc ~]# docker exec -it zookeeper whoami
root
[root@xdhuxc ~]# cat /etc/passwd | grep 1000
[root@xdhuxc ~]# docker exec -it zookeeper id
uid=0(root) gid=0(root)
```
则进程的用户在容器内和容器外都是 root，它可以在容器内对宿主机上的 /bin 目录做任意操作，非常不安全。

docker 1.10 中引入的 user 命名空间就可以让容器有一个“假”的 root 用户，它在容器内是 root，在容器外是一个非 root 用户。也就是说，user 命名空间实现了宿主机 users 和 容器内 users 之间的映射。

docker 安装时创建了一个用户和用户组都叫做 dockremap，容器内的 root 用户映射到宿主机的这个 dockremap 用户上。
```
[root@xdhuxc ~]# cat /etc/passwd | grep dockremap
dockremap:x:992:988::/home/dockremap:/bin/false
```

启用步骤：
1、创建 /etc/subuid 和 /etc/subgid 文件，并分别向其中写入如下内容：
```
echo "dockremap:165536:65536" > /etc/subuid
echo "dockremap:165536:65536" > /etc/subgid
```

2、修改 /usr/lib/systemd/system/docker.service 文件，在启动参数中添加 "--userns-remap=dockremap"。

2、重启 docker，此时 docker 进程为：
```
[root@xdhuxc ~]# systemctl status docker -l
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-07-11 08:14:02 CST; 14s ago
     Docs: https://docs.docker.com
 Main PID: 2487 (dockerd)
   Memory: 22.4M
   CGroup: /system.slice/docker.service
           ├─2487 /usr/bin/dockerd --userns-remap=dockremap --storage-driver=overlay2 --insecure-registry 0.0.0.0/0 --exec-opt native.cgroupdriver=systemd
           └─2494 docker-containerd --config /var/run/docker/containerd/containerd.toml
```
注意，启用新的 user namespaces后，会创建新的镜像和容器存储目录，原有镜像和容器信息依然存在，但是在当前看不到。


3、创建一个容器
```

```

4、查看进程在容器内外的用户：
```
```





查看 /etc/subuid 文件和 /etc/subgid 文件，可以看到 dockermap 用户在宿主机上的 uid 和 gid：
```
```

再查看文件 /proc/1726/uid_map，它表示了容器内外用户的映射关系，即将宿主机上的 231072 用户映射为容器内的 0 （即 root）用户。

现在，当我们试图修改宿主机上的 /bin 目录时，就会提示权限不足了：
```
```
通过使用 user 命名空间，使得容器内的进程运行在非 root 用户，这样就成功地限制了容器内进程的权限。


#### network 命名空间
默认情况下，当 docker 实例被创建出来后，使用 `ip netns` 命令无法看到容器实例对应的 network 命名空间，这是因为 ip netns 命令 是从 /var/run/netns 目录中读取内容的。

操作步骤：
1、找到容器的主进程 ID
```
[root@xdhuxc ~]# docker inspect --format '{{.State.Pid}}' zookeeper
3300
```

2、创建 /var/run/netns 目录及符号链接
``` 
[root@xdhuxc ~]# mkdir /var/run/netns
[root@xdhuxc ~]# ln -s /proc/3300/ns/net /var/run/netns/zookeeper 
```
3300 是 zookeeper 容器在宿主机上的进程ID。
```
[root@xdhuxc ~]# ps -ef | grep -v grep | grep 3300
1000      3300  3284  0 08:37 ?        00:01:10 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
```
而 1000 是容器内的 zookeeper 用户的用户ID
```
bash-4.4# cat /etc/passwd|grep 1000
zookeeper:x:1000:1000:Linux User,,,:/home/zookeeper:
```
说明在创建容器的时候，docker 自动在 /proc 目录下为每一个容器以其进程ID为目录名创建了存放包括cpu、内存、io、命名空间等各种资源的目录，删除容器时会自动删除该目录。
```
[root@xdhuxc 3300]# pwd
/proc/3300
[root@xdhuxc 3300]# ls
attr       clear_refs       cpuset   fd       limits     mem         net        oom_score      projid_map  setgroups  statm    timers
autogroup  cmdline          cwd      fdinfo   loginuid   mountinfo   ns         oom_score_adj  root        smaps      status   uid_map
auxv       comm             environ  gid_map  map_files  mounts      numa_maps  pagemap        sched       stack      syscall  wchan
cgroup     coredump_filter  exe      io       maps       mountstats  oom_adj    personality    sessionid   stat       task
```

3、此时，可以使用 ip netns 命令了。
```
[root@xdhuxc ~]# ip netns
zookeeper (id: 0)
[root@xdhuxc ~]# ip netns exec zookeeper ip addr           
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
237: eth0@if238: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
也就是容器内的网桥信息
```
[root@xdhuxc ~]# docker exec -it zookeeper ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
237: eth0@if238: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

#### docker run 命令中 namespace 相关参数
docker run 命令中如下几个参数和 namespace 相关：
* --ipc string 指定使用的 IPC 命名空间
* --pid string 指定使用的 PID 命名空间
* --userns string 指定使用的 User 命名空间
* --uts string 指定使用的 UTS 命名空间

##### --userns
--userns：指定容器使用的 user 命名空间
* "host"：使用 docker 所在宿主机的 user 命名空间。
* ""：使用由 `--userns-remap` 指定的 docker daemon user 命名空间。

可以在启用了 user 命名空间的情况下，强制某个容器运行在宿主机 user 命名空间之中：
```
[root@xdhuxc xdhuxc]# ps -ef|grep zookeeper
1000      9575  9559  0 19:47 ?        00:00:01 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
root      9910  8341  0 19:51 pts/1    00:00:00 grep --color=auto zookeeper
[root@xdhuxc xdhuxc]# whoami
root
[root@xdhuxc xdhuxc]# docker exec -it zookeeper whoami
root
```
否则默认的话，就会运行在特定的 user 命名空间之中了。

#### --pid
可以指定容器使用 docker 宿主机 pid 命名空间。这样，在容器中的进程，可以看到宿主机上的所有进程。注意，此时不能启用 user 命名空间。
```
[root@xdhuxc xdhuxc]# docker run -d --name zookeeper --pid host --userns host zookeeper:latest
5b847849106c5d597e33f9f3c346ab2a5a5638f6978d7a3ac6c6e1f2a61e0971
[root@xdhuxc xdhuxc]# docker exec -it zookeeper apk add procps
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
(1/3) Installing libintl (0.19.8.1-r1)
(2/3) Installing libproc (3.3.12-r3)
(3/3) Installing procps (3.3.12-r3)
Executing busybox-1.27.2-r11.trigger
OK: 91 MiB in 62 packages
[root@xdhuxc xdhuxc]# docker exec -it zookeeper ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Jun07 ?        00:01:36 /usr/lib/systemd/systemd --switc # 删除了若干与此无关的行
root      3144     1  0 00:37 ?        00:03:34 /usr/bin/dockerd --storage-drive
root      3151  3144  0 00:37 ?        00:02:28 docker-containerd --config /var/
root      3420  3151  0 00:39 ?        00:00:00 docker-containerd-shim -namespac
999       3436  3420  0 00:39 ?        00:00:00 /bin/sh /usr/lib/rabbitmq/bin/ra
999       3549  3436  0 00:39 ?        00:00:00 /usr/lib/erlang/erts-9.3.3/bin/e
999       3626  3436  0 00:39 ?        00:03:05 /usr/lib/erlang/erts-9.3.3/bin/b
999       3738  3626  0 00:39 ?        00:00:01 erl_child_setup 65536
999       3790  3738  0 00:39 ?        00:00:00 inet_gethost 4
999       3791  3790  0 00:39 ?        00:00:00 inet_gethost 4
993       5460     1  0 Jun30 ?        00:00:00 nginx: worker process
993       5461     1  0 Jun30 ?        00:00:00 nginx: worker process
993       5462     1  0 Jun30 ?        00:00:00 nginx: worker process
993       5463     1  0 Jun30 ?        00:00:00 nginx: worker process
root     10185  3151  0 11:54 ?        00:00:00 docker-containerd-shim -namespac
zookeep+ 10201 10185  4 11:54 ?        00:00:01 /usr/lib/jvm/java-1.8-openjdk/jr
root     10323  8341  0 11:54 ?        00:00:00 docker exec -it zookeeper ps -ef
root     10340 10185  0 11:54 pts/0    00:00:00 ps -ef
root     10347 10185  0 11:54 ?        00:00:00 docker-runc --root /var/run/dock
```

#### --uts
可以使容器使用 docker 宿主机的 uts 命名空间。此时，最明显的是，容器的 hostname 和 docker 宿主机的 hostname 是相同的。

一般情况下，创建的 docker 容器的 hostname 为：
```
[root@xdhuxc ~]# docker run -d -v /bin:/host/bin --name nginx nginx:latest
35ec0c186dfc6746b343f784bfeafd87e75c8eb256a55f0d7fe91db73522f4a9
[root@xdhuxc ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
35ec0c186dfc        nginx:latest        "nginx -g 'daemon of…"   5 seconds ago       Up 4 seconds        80/tcp              nginx
[root@xdhuxc ~]# docker exec -it nginx bash
root@35ec0c186dfc:/# hostname
35ec0c186dfc                                # 随机生成的主机名
root@35ec0c186dfc:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	35ec0c186dfc                    # docker 随机生成的主机名，与主机名一致
root@35ec0c186dfc:/# domainname
(none)
root@35ec0c186dfc:/# 
```
为 docker 创建的一个随机的字符串。

为新启动的容器应用宿主机的 uts 命名空间
```
[root@xdhuxc xdhuxc]# docker run -d --name zookeeper --uts host zookeeper:latest
97cb004b489cbf5e0073a14931953c6b892cbdd4da9df4c06fa7e219727e6780
[root@xdhuxc xdhuxc]# docker exec -it zookeeper hostname
xdhuxc                         # zookeeper 容器的主机名
[root@xdhuxc xdhuxc]# docker exec -it zookeeper cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	97cb004b489c       # docker 为每一个容器随机生成的主机名
[root@xdhuxc xdhuxc]# hostname
xdhuxc                         # 宿主机的主机名
```
但是，却存在这种情况，不知道是什么含义及原因。
```
[root@xdhuxc xdhuxc]# docker inspect --format='{{.HostConfig.UTSMode}}' zookeeper
host
[root@xdhuxc xdhuxc]# docker inspect --format='{{.Config.Hostname}}' zookeeper
97cb004b489c
```
总之，docker 守护进程为每个容器都创建了六种 namespaces 的实例，使得容器中的进程都处于一种隔离的运行环境之中。

### Linux control groups
Linux Cgroup 可以让我们为系统中所运行的任务（进程）的用户定义组群分配资源，比如 CPU 时间、系统内存、网络带宽或者这些资源的组合。我们可以监控配置的cgroup，拒绝 cgroup 访问某些资源，甚至在运行的系统中动态配置 cgroup。可以将 control groups 理解为 controller （system resource ）（for）（process）groups，也就是说 cgroup 以一组进程为目标进行系统资源分配和控制。

cgroup 主要提供如下功能：
* Resource Limitation：限制资源使用，比如，内存使用上限，文件系统的缓存限制等。
* Prioritization：优先级控制，比如，CPU 利用和磁盘 I/O 吞吐。
* Accounting：一些审计或统计，主要目的是为了计费。
* Control：挂起进程，恢复执行进程。

使用 cgroup，系统管理员可以更具体地控制对系统资源的分配、优先顺序、拒绝、管理和监控，可以更好地根据任务和用户分配硬件资源，提高总体效率。

在实际的使用中，系统管理员一般会利用 cgroup 做以下几方面工作：
* 隔离一个进程集合（例如，nginx 的所有进程），并限制它们所消耗的资源，比如绑定 CPU 的核。
* 为这组进程分配其足够使用的内存。
* 为这组进程分配相应的网络带宽和磁盘存储限制。
* 限制访问某些设备，通过设置设备的白名单来实现。

在 linux 系统中，一切皆文件。为了方便用户使用，Linux 也将 cgroup 实现成了文件系统。
```
[root@xdhuxc xdhuxc]# mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/net_cls type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
[root@xdhuxc xdhuxc]# lssubsys -m
cpuset /sys/fs/cgroup/cpuset
cpu,cpuacct /sys/fs/cgroup/cpu,cpuacct
memory /sys/fs/cgroup/memory
devices /sys/fs/cgroup/devices
freezer /sys/fs/cgroup/freezer
net_cls /sys/fs/cgroup/net_cls
blkio /sys/fs/cgroup/blkio
perf_event /sys/fs/cgroup/perf_event
hugetlb /sys/fs/cgroup/hugetlb
[root@xdhuxc xdhuxc]# ll /sys/fs/cgroup/
total 0
drwxr-xr-x 5 root root  0 Jul 11 20:05 blkio
lrwxrwxrwx 1 root root 11 Jun  7 15:40 cpu -> cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jun  7 15:40 cpuacct -> cpu,cpuacct
drwxr-xr-x 5 root root  0 Jul 11 20:05 cpu,cpuacct
drwxr-xr-x 4 root root  0 Jun  7 15:40 cpuset
drwxr-xr-x 5 root root  0 Jul 11 20:05 devices
drwxr-xr-x 4 root root  0 Jun  7 15:40 freezer
drwxr-xr-x 4 root root  0 Jun  7 15:40 hugetlb
drwxr-xr-x 5 root root  0 Jul 11 20:05 memory
drwxr-xr-x 4 root root  0 Jun  7 15:40 net_cls
drwxr-xr-x 4 root root  0 Jun  7 15:40 perf_event
drwxr-xr-x 5 root root  0 Jul 11 20:05 systemd
```
可以看到 /sys/fs/cgroup 目录中有若干个子目录，可以认为这些都是受 cgroup 控制的资源以及这些资源的信息。
* blkio：该子系统为块设备设定输入/输出限制，比如物理设备（磁盘、固态硬盘、USB等等）。
* cpu：该子系统使用调度程序提供对 CPU 的 cgroup 任务访问。
* cpuacct：该子系统自动生成 cgroup 中任务所使用的 CPU 报告。
* cpuset：该子系统为 cgroup 中的任务分配独立 CPU（在多核系统） 和内存节点。
* devices：该子系统可允许或者拒绝 cgroup 中的任何访问设备。
* freezer：该子系统挂起或者恢复 cgroup 中的任务。
* memory：该子系统设定 cgroup 中任务使用的内存限制，并自动生成内存资源使用报告。
* net_cls：该子系统使用等级识别符（classid）标记网络数据包，可允许 Linux 流量控制程序（tc）识别从具体 cgroup 中生成的数据包。
* net_prio：该子系统用来设计网络流量的优先级。
* hugetlb：该子系统主要针对于 HugeTLB 系统进行限制，这是一个大页文件系统。

在 CentOS 7.2.1511 系统中，默认看不到 net_prio 目录

#### 通过 cgroup 限制进程的 CPU
编写如下 C 语言程序
```
[root@xdhuxc xdhuxc]# cat xdhuxc.cpp 
int main(void){
	int i = 0;
	for( ; ; ) {
		i++;
	}
	return 0;
}
```
编译并运行
```
[root@xdhuxc xdhuxc]# gcc xdhuxc.cpp -o xdhuxc.out
[root@xdhuxc xdhuxc]# ./xdhuxc.out 
```
使用 top 命令查看 CPU 占用
```
Tasks: 131 total,   2 running, 129 sleeping,   0 stopped,   0 zombie
%Cpu(s): 23.2 us,  0.1 sy,  0.0 ni, 76.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8010904 total,  4122568 free,   331844 used,  3556492 buff/cache
KiB Swap:  2096124 total,  2088680 free,     7444 used.  7014016 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
18110 root      20   0    4164    352    280 R  99.7  0.0   2:25.39 xdhuxc.out 
```

接下来，我们使用 cgroup 限制其 CPU 占用，操作如下
```
[root@xdhuxc xdhuxc]# mkdir /sys/fs/cgroup/cpu/xdhuxc
[root@xdhuxc xdhuxc]# cd /sys/fs/cgroup/cpu/xdhuxc
[root@xdhuxc xdhuxc]# ls
cgroup.clone_children  cpuacct.stat          cpu.cfs_period_us  cpu.rt_runtime_us  notify_on_release
cgroup.event_control   cpuacct.usage         cpu.cfs_quota_us   cpu.shares         tasks
cgroup.procs           cpuacct.usage_percpu  cpu.rt_period_us   cpu.stat
[root@xdhuxc xdhuxc]# cat cpu.cfs_quota_us 
-1
[root@xdhuxc xdhuxc]# echo 20000 > cpu.cfs_quota_us 
[root@xdhuxc xdhuxc]# cat cpu.cfs_quota_us 
20000
[root@xdhuxc xdhuxc]# echo 18110 > tasks
```
说明：
* mkdir /sys/fs/cgroup/cpu/xdhuxc：创建一个 CPU 控制组 xdhuxc，创建 xdhuxc 之后，可以看到目录下自动创建了相关的文件，这些文件都是伪文件。
* cpu.cfs_period_us：CPU 分配的时间周期，以微秒计算，默认为：100000.
* cpu.cfs_quota_us：表示该 control group 限制占用的时间（以微秒计算），默认为：-1，表示不限制。如果设置为2000，表示占用20000/100000=20%的CPU。


在多核情况下，看到的值会不一样。另外，cfs_quota_us 也是可以大于 cfs_period_us 的，这主要是对于多核情况。有 n 个核时，一个控制组中的进程自然最多就能用到 n 倍的 cpu 时间。

这两个值在 cgroups 层次中是有限制的，下层的资源不能超过上层。具体的说，就是下层的 cpu.cfs_period_us 值不能小于上层的值，cpu.cfs_quota_us 值不能大于上层的值。

http://xiezhenye.com/2013/10/%E7%94%A8-cgroups-%E7%AE%A1%E7%90%86-cpu-%E8%B5%84%E6%BA%90.html

注意：此种方式对C，shell，python起作用，对java似乎不起作用。

然后再查看 CPU 占用
```
Tasks: 131 total,   2 running, 129 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.6 us,  0.3 sy,  0.0 ni, 95.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.1 st
KiB Mem :  8010904 total,  4123224 free,   331196 used,  3556484 buff/cache
KiB Swap:  2096124 total,  2088680 free,     7444 used.  7014740 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
18110 root      20   0    4164    352    280 R  19.9  0.0   8:25.99 xdhuxc.out          
```
它占用的 CPU 几乎就是 20%，也就是我们预设的阈值。这说明通过上面的操作，成功地将这个进程运行所占用的 CPU 资源限制在某个阈值范围内了。

此时加入另一个 xdhuxc.out
```
Tasks: 132 total,   3 running, 129 sleeping,   0 stopped,   0 zombie
%Cpu(s): 27.8 us,  0.2 sy,  0.0 ni, 71.8 id,  0.2 wa,  0.0 hi,  0.0 si,  0.1 st
KiB Mem :  8010904 total,  4123328 free,   331072 used,  3556504 buff/cache
KiB Swap:  2096124 total,  2088680 free,     7444 used.  7014856 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
19016 root      20   0    4164    352    280 R 100.0  0.0   1:10.95 xdhuxc.out                           
18110 root      20   0    4164    352    280 R  20.0  0.0   9:07.16 xdhuxc.out       
```
将此 xdhuxc.out 进程的 PID 加入 tasks 文件中，则这两个进程会共享设定的 CPU 限制
```
[root@xdhuxc xdhuxc]# echo 19016 >> tasks 
```
使用 top 命令查看两个 xdhuxc.out 进程的 CPU 占用
```
Tasks: 132 total,   3 running, 129 sleeping,   0 stopped,   0 zombie
%Cpu(s):  6.3 us,  1.6 sy,  0.0 ni, 91.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.3 st
KiB Mem :  8010904 total,  4123656 free,   330696 used,  3556552 buff/cache
KiB Swap:  2096124 total,  2088680 free,     7444 used.  7015244 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                               
18110 root      20   0    4164    352    280 R  10.7  0.0   9:28.38 xdhuxc.out                                                                                      
19016 root      20   0    4164    352    280 R   9.3  0.0   2:16.69 xdhuxc.out                 
```
可见，两个进程比较平均地分享了 CPU 时间。

#### 通过 cgroup 限制进程的内存
类似的，我们针对它占用的内存做如下操作：
```
[root@xdhuxc memory]# mkdir /sys/fs/cgroup/memory/xdhuxc
[root@xdhuxc memory]# cd /sys/fs/cgroup/memory/xdhuxc
[root@xdhuxc xdhuxc]# cat memory.limit_in_bytes 
9223372036854775807
[root@xdhuxc xdhuxc]# pwd
/sys/fs/cgroup/memory/xdhuxc
[root@xdhuxc xdhuxc]# echo 128M > memory.limit_in_bytes 
[root@xdhuxc xdhuxc]# echo 18110 > tasks
```
上面的步骤会把进程ID为 18110 的进程占用的内存阈值设置为 128M，超过的话，它会被杀掉。

#### 通过 cgroup 限制进程的 I/O
运行命令：
```
sudo dd if=/dev/sda of=/dev/null
```
使用 iotop 命令查看 IO（此时磁盘在快速转动），此时，其读速度为：3.21G/s：
```
Total DISK READ :      38.15 M/s | Total DISK WRITE :       0.00 B/s
Actual DISK READ:      38.15 M/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                                                     
20322 be/4 root          5.18 G      0.00 B  0.00 % 75.31 % dd if=/dev/sda of=/dev/null
28167 be/0 root          2.35 G      0.00 B  0.00 %  4.58 % [loop0]
```
接着做下面的操作：
```
[root@xdhuxc ~]# mkdir /sys/fs/cgroup/blkio/xdhuxc
[root@xdhuxc ~]# cd /sys/fs/cgroup/blkio/xdhuxc
[root@xdhuxc xdhuxc]# echo "253:0 1048576" > blkio.throttle.read_bps_device # 253:0 对应主设备号和副设备号，可以通过 ls -l /dev/sda 查看
[root@xdhuxc xdhuxc]# echo 20322 > tasks 
```
再使用 iotop 命令查看，读速度已经降到了 1M/s 以下。
```
20322 be/4 root      993.36 K/s    0.00 B/s  0.00 %  0.00 % dd if=/dev/sda of=/dev/null  
```
似乎对 /dev/vda 设备的IO速度控制不起作用。

可能只对sda设备起作用，也可能用法不对。

#### 术语
cgroups 的术语包括：
* 任务：Tasks，就是系统的一个进程。
* 控制组：Control Group，一组按照某种标准划分的进程，其表示了某进程组。cgroup 中的资源控制都是以控制组为单位实现。一个进程可以加入到某个控制组。而资源的限制是定义在这个组上，简单来说，cgroup就是一个目录带一系列的可配置文件。
* 层级：Hierarchy，控制组可以组织成hierarchy的形式，即一棵控制组的树形结构（目录结构），控制组树上的子节点继承父节点的属性，简单来说，hierarchy就是在一个或多个子系统上的cgroup目录树。
* 子系统：Subsystem，一个子系统就是一个资源控制器，比如CPU子系统就是控制CPU时间分配的一个控制器。子系统必须附加到一个层级上才能起作用，一个子系统附加到某个层级以后，这个层级上的所有控制组群都受到这个子系统的控制。cgroup的子系统可以有很多，也在不断地增加中。

### docker 使用 cgroup 限制容器进程资源的使用
环境信息：
```
[root@xdhuxc ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 1
Server Version: 17.12.1-ce
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 9b55aab90508bd389d7654c4baf173a981477d55
runc version: 9f9c96235cc97674e935002fc3d78361b696a69e
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-693.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 976.3MiB
Name: xdhuxc
ID: JC2S:MKCL:G33Y:CTUN:T5EX:A4J2:HLOM:G4DC:FPX7:ERZL:OF7G:34HV
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 0.0.0.0/0
 127.0.0.0/8
Live Restore Enabled: false

```

docker 安装时，会在 /sys/fs/cgroup/ 目录下的各个资源目录下生成以 docker 为名字的目录，即docker控制组。
```
[root@xdhuxc docker]# pwd
/sys/fs/cgroup/cpu,cpuacct/docker
[root@xdhuxc docker]# ls
cgroup.clone_children  cgroup.procs  cpuacct.usage         cpu.cfs_period_us  cpu.rt_period_us   cpu.shares  notify_on_release
cgroup.event_control   cpuacct.stat  cpuacct.usage_percpu  cpu.cfs_quota_us   cpu.rt_runtime_us  cpu.stat    tasks
```
默认情况下，docker 每启动一个容器时，会在 /sys/fs/cgroup/cpu/docker/ 目录下，生成以 容器ID 为名字的目录，即当前容器的控制组，如下所示：
```
[root@xdhuxc ~]# docker run -d --name=zookeeper zookeeper:latest
c86c1335154d50b69a1133ef92baf56705bfdf76e7f2b7ed5d58279c764c1b9a
[root@xdhuxc ~]# cd /sys/fs/cgroup/cpu/docker
[root@xdhuxc docker]# ll
total 0
drwxr-xr-x 2 root root 0 Jul 15 10:03 c86c1335154d50b69a1133ef92baf56705bfdf76e7f2b7ed5d58279c764c1b9a
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.clone_children
--w--w--w- 1 root root 0 Jul 15 09:39 cgroup.event_control
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.procs
-r--r--r-- 1 root root 0 Jul 15 09:39 cpuacct.stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpuacct.usage
-r--r--r-- 1 root root 0 Jul 15 09:39 cpuacct.usage_percpu
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.rt_period_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.shares
-r--r--r-- 1 root root 0 Jul 15 09:39 cpu.stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 notify_on_release
-rw-r--r-- 1 root root 0 Jul 15 09:39 tasks
```

此时，cpu.cfs_quota_us 的内容为：-1，表示默认情况下并没有限制容器的 CPU 使用。
```
[root@xdhuxc c86c1335154d50b69a1133ef92baf56705bfdf76e7f2b7ed5d58279c764c1b9a]# pwd
/sys/fs/cgroup/cpu/docker/c86c1335154d50b69a1133ef92baf56705bfdf76e7f2b7ed5d58279c764c1b9a
[root@xdhuxc c86c1335154d50b69a1133ef92baf56705bfdf76e7f2b7ed5d58279c764c1b9a]# cat cpu.cfs_quota_us 
-1
```

在容器被 stopped 后，该目录会被删除。

接下来创建一个带资源限制的 zookeeper 容器
```
[root@xdhuxc docker]# pwd
/sys/fs/cgroup/cpu,cpuacct/docker
[root@xdhuxc docker]# docker run -d --name zookeeper --cpu-quota 25000 --cpu-period 150000 --cpu-shares 30 zookeeper:latest
61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6
[root@xdhuxc docker]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                          NAMES
61952fccaff9        zookeeper:latest    "/docker-entrypoint.…"   About a minute ago   Up About a minute   2181/tcp, 2888/tcp, 3888/tcp   zookeeper
[root@xdhuxc docker]# ll
total 0
drwxr-xr-x 2 root root 0 Jul 15 10:20 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.clone_children
--w--w--w- 1 root root 0 Jul 15 09:39 cgroup.event_control
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.procs
-r--r--r-- 1 root root 0 Jul 15 09:39 cpuacct.stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpuacct.usage
-r--r--r-- 1 root root 0 Jul 15 09:39 cpuacct.usage_percpu
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.rt_period_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 Jul 15 09:39 cpu.shares
-r--r--r-- 1 root root 0 Jul 15 09:39 cpu.stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 notify_on_release
-rw-r--r-- 1 root root 0 Jul 15 09:39 tasks
[root@xdhuxc docker]# cd 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6/
[root@xdhuxc 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6]# ls
cgroup.clone_children  cgroup.procs  cpuacct.usage         cpu.cfs_period_us  cpu.rt_period_us   cpu.shares  notify_on_release
cgroup.event_control   cpuacct.stat  cpuacct.usage_percpu  cpu.cfs_quota_us   cpu.rt_runtime_us  cpu.stat    tasks
[root@xdhuxc 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6]# cat cpu.cfs_quota_us 
25000
[root@xdhuxc 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6]# cat cpu.cfs_period_us 
150000
[root@xdhuxc 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6]# cat cpu.shares 
30
```
以上值正是我们在启动容器时设置的资源值。

docker 会将容器中的进程的 ID 加入到各个资源对应的 tasks 文件中。
在宿主机上查看进程
```
[root@xdhuxc docker]# ps -ef | grep -v grep | grep zookeeper
xdhuxc     2890   2879  0 10:20 ?        00:00:00 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
```
tasks 文件中的内容为：
```
[root@xdhuxc 61952fccaff947c1fab3f724e349cd8c3562b92d74e053826830b3d99a3060b6]# cat tasks 
2890                # 正是宿主机上zookeeper进程的PID
2925
2926
2927
2928
2929
2930
2931
2932
2933
2934
2935
2936
2937
2938
```

表明 docker 是通过上面的机制来使用 cgroup 对容器的 CPU 使用进行限制的。


类似的，可以通过 docker run 中 mem 相关的参数对容器的内存使用进行限制：
```
    --kernel-memory bytes            Kernel memory limit
    --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
-m, --memory bytes                   Memory limit
    --memory-reservation bytes       Memory soft limit
    --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
    --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
```
例如，运行如下命令
```
[root@xdhuxc docker]# docker run -d --name zookeeper --blkio-weight 100 --memory 50M --cpu-quota 25000 --cpu-period 150000 --cpu-shares 30 zookeeper:latest
bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
[root@xdhuxc docker]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                          NAMES
bf59ef7c32f2        zookeeper:latest    "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds        2181/tcp, 2888/tcp, 3888/tcp   zookeeper
[root@xdhuxc docker]# ps -ef | grep -v grep | grep zookeeper
xdhuxc     3101   3090  0 10:53 ?        00:00:02 /usr/lib/jvm/java-1.8-openjdk/jre/bin/java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /zookeeper-3.4.12/bin/../build/classes:/zookeeper-3.4.12/bin/../build/lib/*.jar:/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /conf/zoo.cfg
[root@xdhuxc docker]# ps -T -p 3101                # 查看进程ID为3101的进程中的线程
   PID   SPID TTY          TIME CMD
  3101   3101 ?        00:00:00 java
  3101   3143 ?        00:00:00 java
  3101   3144 ?        00:00:01 java
  3101   3145 ?        00:00:00 java
  3101   3146 ?        00:00:00 java
  3101   3147 ?        00:00:00 java
  3101   3148 ?        00:00:00 java
  3101   3149 ?        00:00:00 java
  3101   3150 ?        00:00:00 java
  3101   3151 ?        00:00:00 java
  3101   3152 ?        00:00:00 java
  3101   3157 ?        00:00:00 java
  3101   3158 ?        00:00:00 java
  3101   3159 ?        00:00:00 java
  3101   3160 ?        00:00:00 java
[root@xdhuxc docker]# pwd
/sys/fs/cgroup/memory/docker
[root@xdhuxc docker]# ll
total 0
drwxr-xr-x 2 root root 0 Jul 15 10:53 bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.clone_children
--w--w--w- 1 root root 0 Jul 15 09:39 cgroup.event_control
-rw-r--r-- 1 root root 0 Jul 15 09:39 cgroup.procs
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.failcnt
--w------- 1 root root 0 Jul 15 09:39 memory.force_empty
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.failcnt
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.limit_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.max_usage_in_bytes
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.slabinfo
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.tcp.failcnt
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.tcp.limit_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.tcp.usage_in_bytes
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.kmem.usage_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.limit_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.max_usage_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.memsw.failcnt
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.memsw.limit_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.memsw.max_usage_in_bytes
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.memsw.usage_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.move_charge_at_immigrate
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.numa_stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.oom_control
---------- 1 root root 0 Jul 15 09:39 memory.pressure_level
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.soft_limit_in_bytes
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.stat
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.swappiness
-r--r--r-- 1 root root 0 Jul 15 09:39 memory.usage_in_bytes
-rw-r--r-- 1 root root 0 Jul 15 09:39 memory.use_hierarchy
-rw-r--r-- 1 root root 0 Jul 15 09:39 notify_on_release
-rw-r--r-- 1 root root 0 Jul 15 09:39 tasks
[root@xdhuxc docker]# cd bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da/
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# ls
cgroup.clone_children  memory.kmem.failcnt             memory.kmem.tcp.limit_in_bytes      memory.max_usage_in_bytes        memory.move_charge_at_immigrate  memory.stat            tasks
cgroup.event_control   memory.kmem.limit_in_bytes      memory.kmem.tcp.max_usage_in_bytes  memory.memsw.failcnt             memory.numa_stat                 memory.swappiness
cgroup.procs           memory.kmem.max_usage_in_bytes  memory.kmem.tcp.usage_in_bytes      memory.memsw.limit_in_bytes      memory.oom_control               memory.usage_in_bytes
memory.failcnt         memory.kmem.slabinfo            memory.kmem.usage_in_bytes          memory.memsw.max_usage_in_bytes  memory.pressure_level            memory.use_hierarchy
memory.force_empty     memory.kmem.tcp.failcnt         memory.limit_in_bytes               memory.memsw.usage_in_bytes      memory.soft_limit_in_bytes       notify_on_release
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# cat memory.limit_in_bytes 
52428800                   # 正是我们设置的50M的限额
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# cat tasks 
3101                       # zookeeper 容器进程在宿主机上的进程ID。
3143                       # 以下是 zoookeeper 容器进程所起的线程ID。
3144
3145
3146
3147
3148
3149
3150
3151
3152
3157
3158
3159
3160
```
查看 IO 限制
```
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# pwd
/sys/fs/cgroup/blkio/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# ls
blkio.io_merged                   blkio.io_serviced                blkio.leaf_weight                blkio.throttle.io_serviced        blkio.time_recursive   notify_on_release
blkio.io_merged_recursive         blkio.io_serviced_recursive      blkio.leaf_weight_device         blkio.throttle.read_bps_device    blkio.weight           tasks
blkio.io_queued                   blkio.io_service_time            blkio.reset_stats                blkio.throttle.read_iops_device   blkio.weight_device
blkio.io_queued_recursive         blkio.io_service_time_recursive  blkio.sectors                    blkio.throttle.write_bps_device   cgroup.clone_children
blkio.io_service_bytes            blkio.io_wait_time               blkio.sectors_recursive          blkio.throttle.write_iops_device  cgroup.event_control
blkio.io_service_bytes_recursive  blkio.io_wait_time_recursive     blkio.throttle.io_service_bytes  blkio.time                        cgroup.procs
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# cat blkio.weight
100                        # 正是我们设置的 blkio-weight 的值。
[root@xdhuxc bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da]# cat tasks 
3101                       # zookeeper 容器进程在宿主机上的进程ID。
3143                       # 以下是 zoookeeper 容器进程所起的线程ID。
3144
3145
3146
3147
3148
3149
3150
3151
3152
3157
3158
3159
3160
```

目录 cpu 和 目录 cpuacct 为符号链接文件，都链接到了 cpu,cpuacct 目录。
```
[root@xdhuxc cgroup]# pwd
/sys/fs/cgroup
[root@xdhuxc cgroup]# ls
blkio  cpu  cpuacct  cpu,cpuacct  cpuset  devices  freezer  hugetlb  memory  net_cls  perf_event  systemd
[root@xdhuxc cgroup]# ll
total 0
drwxr-xr-x 5 root root  0 Jul  4 08:52 blkio
lrwxrwxrwx 1 root root 11 Jun  7 15:40 cpu -> cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jun  7 15:40 cpuacct -> cpu,cpuacct
drwxr-xr-x 5 root root  0 Jul  4 08:52 cpu,cpuacct
drwxr-xr-x 4 root root  0 Jun  7 15:40 cpuset
drwxr-xr-x 5 root root  0 Jul  4 08:52 devices
drwxr-xr-x 4 root root  0 Jun  7 15:40 freezer
drwxr-xr-x 4 root root  0 Jun  7 15:40 hugetlb
drwxr-xr-x 5 root root  0 Jul  4 08:52 memory
drwxr-xr-x 4 root root  0 Jun  7 15:40 net_cls
drwxr-xr-x 4 root root  0 Jun  7 15:40 perf_event
drwxr-xr-x 5 root root  0 Jul  4 08:52 systemd
```
从上面也可以看出，目前 docker 已经几乎支持了所有的 cgroup 资源，可以限制容器对包括 network、device、cpu 和 memory 在内的资源的使用。
```
[root@xdhuxc cgroup]# pwd
/sys/fs/cgroup
[root@xdhuxc cgroup]# find ./ -name bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./blkio/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./cpuset/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./perf_event/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./freezer/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./cpu,cpuacct/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./pids/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./net_cls,net_prio/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./devices/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./hugetlb/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./memory/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
./systemd/docker/bf59ef7c32f2f9d3878e9ee9f013288031cbcfe9ec03d8efbb075c14150828da
```

#### 限制容器的网络流量
net_cls 和 tc 一起使用可用于限制进程发出的网络包所使用的网络带宽。当使用 cgroup network controll net_cls 后，指定进程发出的所有网络包都会被加一个标签，然后就可以使用其他工具，比如 iptables 或者 traffic controller（TC）来根据网络包上的标签进行流量控制。

关于 classid，它的格式是：0xAAAABBBB，其中，AAAA 是十六进制的主 ID（major number），BBBB 是十六进制的次 ID（minor number）。因此，0x10001 表示 10:1 ，而 0x00010001 表示 1:!

1、首先，在宿主机的网卡 eth0 上做如下设置：
```
tc qdisc del dev eth0 root                         # 删除已有规则
tc qdisc add dev eth0 root handle 10: htb default 12
tc class add dev eth0 parent 10: classid 10:1 htb rate 1500kbit ceil 1500kbit burst 10k    # 限速
tc filter add dev eth0 protocol ip parent 10:0 prio 1 u32 match ip protocol 1 0xff flowid 10:1 # 只处理 ping 参数的网络包
```
其结果是：
* 在网卡 eth0 上创建了一个 HTB root 队列，handle 10: 表示队列句柄，也就是 major number 为 10。
* 创建一个分类 10:1，限制它的发出网络带宽为 80 kbit（千比特每秒）
* 创建一个分类器，将 eth0 上 IP IMCP 协议的 major ID 为 10 的 prio 为 1 的网络流量都分类到 10:1 类别。

2、启动容器
容器启动后，其 init 进程在宿主机上的 PID 就被加入到 tasks 文件中了。
```
```
设置 net_cls classid：
```
echo 0x100001 > net_cls.classid
```
然后在容器中启动一个 ping 进程，其 ID 也被加入到 tasks 文件中了。

3、查看 tc 情况
```
tc -s -d class show dev eth0


```
可以看到 tc 已经在处理 ping 进程产生的数据包了。

再看一下 net_cls 和 ts 合作的限速效果：
```
10488 bytes from 192.168.1.1: icmp_seq=35 ttl=63 time=12.7 ms
10488 bytes from 192.168.1.1: icmp_seq=36 ttl=63 time=15.2 ms
10488 bytes from 192.168.1.1: icmp_seq=37 ttl=63 time=4805 ms
10488 bytes from 192.168.1.1: icmp_seq=38 ttl=63 time=9543 ms
```
其中，
* 后两条说使用的 tc class 规则是 tc class add dev eth0 parent 10: classid 10:1 htb rate 1500kbit ceil 15kbit burst 10k
* 前两条所使用的 tc class 规则是 tc class add dev eth0 parent 10: classid 10:1 htb rate 1500kbit ceil 10Mbit burst 10k 

#### docker run 命令中 cgroup 相关命令

block IO:
```
    --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
    --blkio-weight-device list       Block IO weight (relative device weight) (default [])
    --cgroup-parent string           Optional parent cgroup for the container
```
CPU:
```
    --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
    --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
    --cpu-rt-period int              Limit CPU real-time period in microseconds
    --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
-c, --cpu-shares int                 CPU shares (relative weight)
    --cpus decimal                   Number of CPUs
    --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
    --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
```
Device:
```
    --device list                    Add a host device to the container
    --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
    --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
    --device-read-iops list          Limit read rate (IO per second) from a device (default [])
    --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
    --device-write-iops list         Limit write rate (IO per second) to a device (default [])
```
Memory:
```
    --kernel-memory bytes            Kernel memory limit
-m, --memory bytes                   Memory limit
    --memory-reservation bytes       Memory soft limit
    --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
    --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
```
注意：
* cgroup 只能限制 CPU 的使用，而不能保证 CPU 的使用。也就是说，使用 cpuset-cpus，可以让容器在指定的 CPU 或者核上运行，但是不能确保它独占这些CPU；cpu-shares 是个相对值，只有在 CPU 不够用的时候才起作用，也就是说，当 CPU 够用的时候，每个容器会分到足够的 CPU；当不够用的时候，会按照指定的比重在多个容器之间分配 CPU。
* 对内存来说，cgroup 可以限制容器使用内存的最大值，使用 -m 参数可以设置最多可以使用的内存。



参考资料：

http://www.cnblogs.com/sammyliu/p/5878973.html

https://coolshell.cn/articles/17049.html

https://blog.csdn.net/qinyushuang/article/details/46611709

https://docs.docker.com/config/containers/resource_constraints/#configure-the-realtime-scheduler

http://www.cnblogs.com/sammyliu/p/5886833.html