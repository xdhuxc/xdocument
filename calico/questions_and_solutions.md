### 已经部署好的 calico，需要修改其网段，修改后，连接不通
执行如下命令：
```angular2html
#!/usr/bin/env bash
#重新安装calico (cnm)
# 修改calico地址后，需要删除原来的tunl0等

ip link set tunl0 down
ifconfig|grep "cali\|cni\|flannel"|awk '{printf("ip link delete %s\n", $1)}'|sh
rm -rf /etc/cni/
rm -rf /opt/cni
rm -rf /var/lib/cni
mkdir -p /opt/cni/bin/
iptables -F
systemctl restart iptables
export ETCD_AUTHORITY=10.3.15.189:2379
calicoctl node run --ip=
#安装完后可以docker run -it --net=net1 busybox sh 测试一下ip的ping通性
```