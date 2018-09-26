### 常用命令
1、强制删除 pod
```
kubectl delete pod kubernetes-dashboard-55556f66c5-4zc77 -n kube-system --grace-period=0 --force
```
该命令也可以用于强制删除其他 kubernetes 资源。

2、查看集群信息
```angularjs
[root@k8s-master ~]# kubectl cluster-info
Kubernetes master is running at https://172.20.26.150:6443
KubeDNS is running at https://172.20.26.150:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

3、查看当前上下文
```angularjs
[root@k8s-master ~]# kubectl config current-context
kubernetes-admin@kubernetes
```