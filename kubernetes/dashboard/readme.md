### 免密方式
> 以下假设 kubernetes master 所在机器的IP地址为：172.20.26.150

#### 使用代理方式部署
1、依次执行如下命令部署 dashboard：
```
kubectl create -f kubernetes-dashboard.yaml 
nohup kubectl proxy --address='0.0.0.0' --port=8888 --accept-hosts='^*$' &
```
2、访问地址：
http://172.20.26.150:8888/ui


#### NodePort方式部署 
1、修改 kubernetes-dashboard.yaml 文件，打开 type: NodePort和nodePort: 35588
2、依次执行如下命令部署 dashboard：
```
kubectl create -f kubernetes-dashboard.yaml
```
3、访问地址：http://172.20.26.150:35588

#### 问题
这个无密码登录有个问题：打开容器组的容器面板，右上角有”运行命令”，这里是不能执行的，另外菜单的”设置”不能使用

### kubernetic 客户端方式
1、下载安装[kubernetic](http://kubernetic.com/)

2、执行如下命令生成kubectl配置文件（~/.kube/config）
```
# 创建cluster
kubectl config set-cluster local-server --server=http://172.20.26.150:8080
# 创建默认上下文
kubectl config set-context default-context --cluster=local-server --namespace=kube-system
# 设置默认上下文
kubectl config use-context default-context
# 查看.kube/config文件内容
kubectl config view
```
3、复制config文件内容到windows系统中C:/Users/当前用户目录/.kube/目录（如果没有该目录，手工创建之）下

（无需证书的支持）

4、启动kubernetic

### token登录
1、创建dashboard
```
kubectl delete -f kubernetes-dashboard.yaml 
```
2、获取dashboard 的token
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubernetes-dashboard-token|awk '{print $1}')|grep token:|awk '{print $2}'
```
[root@k8s-master token]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubernetes-dashboard-token|awk '{print $1}')|grep token:|awk '{print $2}'
eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1zNzJwayIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYxNDFmZjcwLWFjMjUtMTFlOC04NDVlLTA2MjY2ODAwMWYxNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.PfAF9nf1SVMLJ-LneTf9QZVgset8UX6pYwgmdds56GM6bJ13-Zqc3CXHUptAJZ8Zxbxy32MgjGFjBr47oocHvMBZn9UkPDcxI9ss8F-PIdmtmuV1bK7L7gXX-4ScIl5UHT3gUHHqJlgV9znbm8ie2WMi0wa_YwUFKMq8arJGngK5BIHV3U1dn5kR7oBZN3owlPh-nWujgB6vBluJ3_NWZPxNqkVm8qxyVa7lkbxK59BKRt9zqRy5Olullxv60fRqf9fuJgfP726848kZkI4HRXdcAW-E4jofCQUlFhuA1CPexrlOmjtcVx5YKxgKf-HJNYHjCMuc7l-rCQl2GEWt3g

3、访问地址：https://172.20.26.150:35888，输入上面的token即可登录。

