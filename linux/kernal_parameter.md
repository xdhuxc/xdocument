内核参数含义：

/etc/sysctl.d/kubernetes.conf
```angular2html
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
vm.swappiness = 0
```

/etc/sysctl.conf 
```angular2html
net.nf_conntrack_max = 1048576
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 3600
net.netfilter.nf_conntrack_tcp_timeout_established = 7200
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_last_ack = 30
net.netfilter.nf_conntrack_tcp_timeout_syn_recv = 60
net.netfilter.nf_conntrack_tcp_timeout_syn_sent = 120
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120

net.ipv4.neigh.default.gc_thresh1=1024
net.ipv4.neigh.default.gc_thresh2=4096
net.ipv4.neigh.default.gc_thresh3=8192
```

### 内核参数含义

#### vm.swappiness 
Linux 会使用硬盘的一部分作为 Swap 分区，用来进行进程的调度。如果内存够大，应当告诉 Linux 不必太多地使用 Swap 分区，可以通过修改 swappiness 的数值实现。

swappiness=0 的时候表示最大限度地使用物理内存，然后才是 swap 空间；

swappiness=100 的时候表示积极地使用 swap 分区，并且把内存上的数据及时地搬运到 swap 空间里面。

在 Linux 中，swappiness 的默认值为：60 


