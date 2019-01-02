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
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce-18.06
systemctl restart docker && systemctl enable docker
```

8、安装 dubeadm 
```angular2html
yum install -y kubeadm 
systemctl restart kubelet && systemctl enable kubelet
```

9、使用 kubeadm
```angular2html
kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
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
