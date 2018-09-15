### 常用命令
1、强制删除 pod
```
kubectl delete pod kubernetes-dashboard-55556f66c5-4zc77 -n kube-system --grace-period=0 --force
```
该命令也可以用于强制删除其他 kubernetes 资源。