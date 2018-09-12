### docker 网络概况


### 四种单节点网络模式

#### bridge 网络模式
docker 容器默认使用 bridge 网络模式，其特点如下：
* 使用一个 linux 网桥，默认为 docker0。
* 使用 veth 对，一端在容器的网络命名空间中，一端在 docker0 上。
* 该模式下，docker 容器不具有一个公有 IP 地址，由于宿主机的 IP 地址与 veth 对的 IP 地址不在同一个网段内。
* docker 采用 NAT 方式，将容器内部的服务监听的端口与宿主机的某一个端口进行绑定，使得宿主机以外的世界可以主动将网络报文发送至容器内部。
* 外界访问容器内的服务时，需要访问宿主机的 IP 地址以及宿主机的端口。
* NAT 模式是在三层网络上的实现手段，因此肯定会影响网络的传输效率。
* 容器拥有独立、隔离的网络栈，让容器和宿主机以外的世界通过 NAT 建立通信。
![image](C:/Users/wanghuan/Desktop/电子书/Docker/docker-bridge.jpg)

iptables 的 SNTA 规则，使得从容器离开去外界的网络包的源IP地址被转换为 docker 宿主机的 IP 地址。
```
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  all  --  172.18.0.0/16        0.0.0.0/0
```

#### host 网络模式
host 模式并没有为容器创建一个隔离的网络环境。之所以称之为 host 模式，是因为该模式下的 docker 容器会和宿主机共享同一个网络命名空间，因此 docker 容器可以和宿主机一样，使用宿主机的 eth0，实现和外界的通信。换句话说，docker 容器的 IP 地址即为宿主机 eth0 的 IP 地址。

![image](C:/Users/wanghuan/Desktop/电子书/Docker/docker-host.jpg)

其特点如下：
* 这种模式下的容器没有隔离的网络命名空间。
* 容器的 IP 地址就是 docker 宿主机的 IP 地址。
* 需要注意容器中服务的端口号不能与 docker 宿主机上已经使用的端口号相冲突。
* host 模式能够与其他模式共存。

#### none 网络模式
网络模式为 none，即不为 docker 容器构造任何网络环境。一旦 docker 容器采用了 none 网络模式，那么容器内部就只能使用 loopback 网络设备，不会再有其他的网络资源。docker 容器的 none 网络模式意味着不给该容器创建任何网络环境，容器只能使用 127.0.0.1 的本机网络。

#### macvlan
https://blog.csdn.net/CloudMan6/article/details/77186548
https://blog.csdn.net/cloudman6/article/details/77338515

https://blog.csdn.net/cloudvtech/article/details/79830887

https://blog.csdn.net/cloudvtech/article/details/79830887


### 多节点 docker 网络
docker 多节点网络模式可以分为两类：
* docker 在 1.19 版本中引入的基于 VxLAN 的对跨节点网络的原生支持。
* 通过插件方式引入的第三方实现方案，比如flannel，calico等。

#### docker 原生 overlay 网络
docker 1.19 版本中增加了对 overlay 网络的原生支持。

docker 支持 consul，etcd和zookeeper 三种分布式的 key-value 存储。

其中，etcd 是一个高可用的分布式 key-value 存储系统，使用 etcd 的场景默认处理的数据都是控制数据，对于应用数据，只推荐数据量很小，但是更新、访问频繁的情况。

1、安装 etcd
在 192.168.91.128 上使用如下命令启动 etcd 容器
```
docker run -d \
    --privileged=true \
    --net=host \
    --restart=always \
    -v /home/xdhuxc/etcd/data:/etcd-data \
    --name etcd \
    quay.io/coreos/etcd:v3.3 \
    /usr/local/bin/etcd \
    --data-dir=/etcd-data --name etcd-single \
    --initial-advertise-peer-urls http://192.168.91.128:2380 --listen-peer-urls http://0.0.0.0:2380 \
    --advertise-client-urls http://192.168.91.128:2379 --listen-client-urls http://0.0.0.0:2379 \
    --initial-cluster etcd-single=http://192.168.91.128:2380
```
修改 docker.service 文件，在 docker 启动参数中添加如下内容：
```
--cluster-store=etcd://192.168.91.128:2379
```
重启 docker 服务
```
systemctl daemon-reload
systemctl restart docker
```

2、在 192.168.91.128 上创建一个 overlay 网络
```
[root@xdhuxc ~]# docker network create --driver overlay net1            # 创建 overlay 网络 net1
0398dac3d6513160ee035c7499332d7dcba8d2fa9643c45488996f628bf8ab91
[root@xdhuxc ~]# docker network ls                                      # 查看当前docker网络
NETWORK ID          NAME                DRIVER              SCOPE
b9149e6e409b        bridge              bridge              local
b9824657dbd0        host                host                local
0398dac3d651        net1                overlay             global
3b216a2f5426        none                null                local
[root@xdhuxc ~]# docker inspect net1
[
    {
        "Name": "net1",
        "Id": "0398dac3d6513160ee035c7499332d7dcba8d2fa9643c45488996f628bf8ab91",
        "Created": "2018-07-15T14:14:48.744993416+08:00",
        "Scope": "global",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
在 192.168.91.130 上配置 etcd 信息到 docker.service 文件中，并重启 docker 服务。

在 192.168.91.130 上，我们也能看到这个网络 net1，这说明通过 etcd 的数据共享，网络数据已经是分布式的了，而不仅仅是本地的。
```
[root@xdhuxc ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2228e2935455        bridge              bridge              local
bddcee3b802a        host                host                local
0398dac3d651        net1                overlay             global
e8a5f558ec10        none                null                local
[root@xdhuxc ~]# docker inspect net1
[
    {
        "Name": "net1",
        "Id": "0398dac3d6513160ee035c7499332d7dcba8d2fa9643c45488996f628bf8ab91",
        "Created": "2018-07-15T14:14:48.744993416+08:00",
        "Scope": "global",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

3、在网络中创建容器
在 192.168.91.128 上基于 net1 创建容器
```
[root@xdhuxc ~]# docker run -d --name over-128 --network net1 zookeeper:latest
adaa369ef21931f04625042949a2f757b9be94a9c67d611a2118e7fecf85f517
[root@xdhuxc ~]# docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                          NAMES
adaa369ef219        zookeeper:latest           "/docker-entrypoint.…"   5 seconds ago       Up 3 seconds        2181/tcp, 2888/tcp, 3888/tcp   over-128
609e50616c43        quay.io/coreos/etcd:v3.3   "/usr/local/bin/etcd…"   13 minutes ago      Up 9 minutes                                       etcd
```
```
[root@xdhuxc ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:25:e5:b2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.91.128/24 brd 192.168.91.255 scope global dynamic ens33
       valid_lft 1785sec preferred_lft 1785sec
    inet6 fe80::63c8:5192:716b:79be/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:54:4e:84:2b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
9: docker_gwbridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP           # 用于 NAT 网络
    link/ether 02:42:a4:44:86:a9 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker_gwbridge
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a4ff:fe44:86a9/64 scope link 
       valid_lft forever preferred_lft forever
11: veth76cddb3@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker_gwbridge state UP 
    link/ether c2:e5:c6:e7:dc:aa brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::c0e5:c6ff:fee7:dcaa/64 scope link 
       valid_lft forever preferred_lft forever
```

docker run -d --name over-130 --network net1 zookeeper:latest
在 192.168.91.130 上基于 net1 创建容器
```
[root@xdhuxc ~]# docker run -d --name over-130 --network net1 zookeeper:latest
516157dec4d555cbdf96783e37a27fa32566b5bcaa375f99aee1a37f7d7e7d33
[root@xdhuxc ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                          NAMES
516157dec4d5        zookeeper:latest    "/docker-entrypoint.…"   5 seconds ago       Up 3 seconds        2181/tcp, 2888/tcp, 3888/tcp   over-130
[root@xdhuxc ~]# 

```

```
[root@xdhuxc ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:58:10:3c brd ff:ff:ff:ff:ff:ff
    inet 192.168.91.130/24 brd 192.168.91.255 scope global dynamic ens38
       valid_lft 1310sec preferred_lft 1310sec
    inet6 fe80::43cc:6988:179b:cd2d/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:fa:be:c4:e2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:faff:febe:c4e2/64 scope link 
       valid_lft forever preferred_lft forever
22: docker_gwbridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP        # 用于 NAT 网络
    link/ether 02:42:27:27:a8:1e brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker_gwbridge
       valid_lft forever preferred_lft forever
    inet6 fe80::42:27ff:fe27:a81e/64 scope link 
       valid_lft forever preferred_lft forever
24: veth06aa5aa@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker_gwbridge state UP 
    link/ether ce:53:38:eb:84:ba brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::cc53:38ff:feeb:84ba/64 scope link 
       valid_lft forever preferred_lft forever
```

进入容器 over-128 中，
```
[root@xdhuxc ~]# docker exec -it over-128 bash
bash-4.4# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP 
    link/ether 02:42:0a:00:00:02 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.2/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
10: eth1@if11: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```

进入 容器 over-130 中，发现它有三块网卡：
```
[root@xdhuxc ~]# docker exec -it over-130 bash
bash-4.4# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
20: eth0@if21: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP 
    link/ether 02:42:0a:00:00:03 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.3/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
23: eth1@if24: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```

其中 eth1@if24 的网络是一个内部的网段，走的还是普通的NAT模式。

eth0@if21 是 overlay 网段上分配的IP地址，它走的是 overlay 网络，其 MTU 是1450，而不是1500。

进一步查看它的路由表，会发现只有同一个 overlay 网络中的容器之间的通信才会通过 eth0@if21，其他所有通信还是走的 eth1。
```
bash-4.4# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.18.0.1      0.0.0.0         UG    0      0        0 eth1
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth1
```
此时的网络拓扑图为：
![image](C:/Users/wanghuan/Desktop/电子书/Docker/docker-network-bridge.jpg)

可见：
* docker 在每个节点上创建了两个 linux 网桥，一个用于 overlay 网络（），一个用于非 overlay 的 NAT 网络（docker_gwbridge）。
* 容器内的到 overlay 网络的其它容器的网络流量走 overlay 网卡，即eth0@if21，其它网络流量走 NAT 网卡，即eth1@if24。
* 当前 docker 创建 vxlan 隧道的ID范围为：256~1000，因而最多可以创建745个网络。因此，本例中的这个 vxlan 隧道使用的 ID 是256。
* docker vxlan 驱动使用 4789 UDP 端口。
```
[root@xdhuxc ~]# netstat -nlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
udp        0      0 0.0.0.0:68              0.0.0.0:*                           3663/dhclient       
udp        0      0 0.0.0.0:4789            0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:59600           0.0.0.0:*                           3663/dhclient       
udp6       0      0 :::31602                :::*                                3663/dhclient       
```
* overlay 网络模型底层需要类似 consul 或 etcd 的 KV 存储系统进行消息同步。
* docker overlay 不使用多播。
* overlay 网络中的容器处于一个虚拟的大二层网络中。

从over-128 ping over-130 不通，据说升级内核到4.20以上就可以了。


### 参考资料
http://www.cnblogs.com/sammyliu/p/5894191.html