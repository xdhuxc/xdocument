### yum 安装

#### 准备工作
1、修改主机名
```angular2html
hostnamectl set-hostname 'k8s-master'
```

2、禁用 SELinux
```angular2html
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

3、修改内核参数
```angular2html
cat <<EOF >  /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF

sysctl --system
```

4、关闭防火墙
```angular2html
systemctl stop firewalld
systemctl disable firewalld
```

5、禁用 swap
```angular2html
swapoff -a
```
修改 /etc/fstab 文件，注释掉 Swap 的自动挂载

6、使用如下 kubernetes repo 源 及 docker 源
```angular2html
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

7、安装 docker 
```angular2html
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo # 配置 docker repo 源
yum list docker-ce.x86_64  --showduplicates |sort -r # 查看最新的 Docker 版本
yum install docker-ce-18.06
systemctl restart docker && systemctl enable docker
```

8、安装 dubeadm 
```angular2html
yum install -y kubeadm 
systemctl restart kubelet && systemctl enable kubelet
```

9、使用 kubeadm
执行命令为：
```angular2html
kubeadm init \
     --kubernetes-version=v1.13.1 \
     --pod-network-cidr=10.244.0.0/16 \
     --apiserver-advertise-address=10.0.2.15
     

mkdir -p $HOME/.kube
\cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

```angular2html
[root@k8s-master ~]# kubeadm init \
>     --kubernetes-version=v1.13.1 \
>     --pod-network-cidr=10.244.0.0/16 \
>     --apiserver-advertise-address=10.0.2.15
[init] Using Kubernetes version: v1.13.1
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "k8s-master" could not be reached
	[WARNING Hostname]: hostname "k8s-master": lookup k8s-master on 10.0.2.3:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 20.512973 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.13" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "k8s-master" as an annotation
[mark-control-plane] Marking the node k8s-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 1n0oej.96h4zyk2j1tnp263
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.0.2.15:6443 --token 1n0oej.96h4zyk2j1tnp263 --discovery-token-ca-cert-hash sha256:927f51ac11f8c874b8b9531d465cedcb8b53028f0aac003be071a2fd2acc60e4

[root@k8s-master ~]# mkdir -p $HOME/.kube
[root@k8s-master ~]# \cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

10、
```angular2html
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES    AGE    VERSION
k8s-master   NotReady   master   5m6s   v1.13.1
```
此时还需要配置网络插件
```angular2html
wget https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/calico.yaml
kubectl apply -f calico.yaml
```
此时再查看 kubernetes 的状态
```angular2html
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   15m   v1.13.1
```





11、重置集群
```angular2html
[root@k8s-master ~]# kubeadm reset
[reset] WARNING: changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] are you sure you want to proceed? [y/N]: y
[preflight] running pre-flight checks
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[reset] stopping the kubelet service
[reset] unmounting mounted directories in "/var/lib/kubelet"
[reset] deleting contents of stateful directories: [/var/lib/etcd /var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]
[reset] deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually.
For example:
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

[root@k8s-master ~]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

#### 问题及解决
1、coredns pod 一直处于 pending 状态，使用 kubectl describe 查看后，错误信息如下：
```angular2html
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     CriticalAddonsOnly
                 node-role.kubernetes.io/master:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  33m                 default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  32m (x3 over 32m)   default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  30m                 default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  26m (x6 over 29m)   default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  14m (x29 over 24m)  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  12m                 default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Warning  FailedScheduling  5m41s               default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
```
这是因为 kubernetes 出于安全考虑，默认情况下无法在 master 节点上部署 pod，由于我们只有一个节点，所以需要取消此限制，执行如下命令：
```angular2html
[root@k8s-master ~]# kubectl taint nodes --all node-role.kubernetes.io/master-
node/k8s-master untainted
[root@k8s-master ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                 READY   STATUS             RESTARTS   AGE
kube-system   coredns-86c58d9df4-gzx76             1/1     Running            0          53m
kube-system   coredns-86c58d9df4-vvbt9             1/1     Running            0          54m
```