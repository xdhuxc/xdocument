### 使用 kubeadmin 安装 kubernetes
```angularjs
1、初始化节点
  kubeadm init --kubernetes-version=v1.9.3 --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.244.128
kubeadmin初始化节点过程：
[root@k8s-master xdhuxc]# kubeadm init --kubernetes-version=v1.9.3 --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.244.128
[init] Using Kubernetes version: v1.9.3
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
	[WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Starting the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.244.128]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 85.522530 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node k8s-master as master by adding a label and a taint
[markmaster] Master k8s-master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 913d4f.e6ea91735f175429
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
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

  kubeadm join --token 913d4f.e6ea91735f175429 192.168.244.128:6443 --discovery-token-ca-cert-hash sha256:acbfa34cd317147aa7e4ab7fe0b93ae4815e746a519420aca4bb8fe6c9afd771

[root@k8s-master xdhuxc]# HOME=~
[root@k8s-master xdhuxc]# mkdir -p $HOME/.kube
[root@k8s-master xdhuxc]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master xdhuxc]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
[root@k8s-master xdhuxc]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   

  
  
2、安装calico
[root@k8s-master xdhuxc]# wget https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
-bash: wget: command not found
[root@k8s-master xdhuxc]# yum install -y wget

kubectl apply -f calico.yaml

kubectl get pods -n kube-system -o wide 

cat /var/log/messages|grep kubelet

kubectl get pods --all-namespaces -o wide

修改
kubelet  --cni-bin-dir=/usr/libexec/cni
/opt/calico/bin 为 /opt/cni/bin

创建目录/opt/calico/bin，将/opt/cni/bin目录下的文件复制到/opt/calico/bin目录下 

--cni-bin-dir=/opt/cni/bin

wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml


kubectl apply -f kubernetes-dashboard.yml

找到kubernetes-dashboard-admin的secret记录
kubectl get secret --all-namespaces | grep kubernetes-dashboard-admin

[root@k8s-master xdhuxc]# kubectl get secret --all-namespaces | grep kubernetes-dashboard-admin
kube-system   kubernetes-dashboard-admin-token-qlssf           kubernetes.io/service-account-token   3         1m


查看secret的token值
kubectl describe secret kubernetes-dashboard-admin-token-b5np5 -n kube-system

kubectl describe secret kubernetes-dashboard-admin-token-qlssf -n kube-system

kubectl get pods --all-namespaces
kubectl get svc --all-namespaces


kubectl get svc --all-namespaces

192.168.244.134:31360

https://192.168.244.128:31360

yum install -y kubernetes

yum install -y etcd
systemctl start etcd
systemctl enable etcd

Environment=HTTPS_PROXY=http://10.3.15.206:8888/
Environment=HTTP_PROXY=http://10.3.15.206:8888/

systemctl daemon-reload
systemctl restart docker

yum install -y https://mirrors.aliyun.com/centos/7.4.1708/virt/x86_64/kubernetes19/kubernetes-kubeadm-1.9.3-1.el7.x86_64.rpm

kubectl apply -f kubernetes-dashboard-rbac-admin.yml 

kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl taint node k8s-master node-role.kubernetes.io/master:master-

kubectl drain k8s-node-1 --delete-local-data --force --ignore-daemonsets

kubeadm init 执行结束之后会输出kubeadm join，在node节点上直接执行即可，如果进行清屏操作，有没有记录，可以使用此命令输出：
kubeadm token create --print-join-command
```
