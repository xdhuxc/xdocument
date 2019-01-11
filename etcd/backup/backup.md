### 含证书的 etcd 备份至 AWS S3 与恢复

方案：使用 coreos 提供的 etcd backup operator，

#### 前提条件
1、已部署含自签名证书的 etcd 集群，基于原生方式部署。

2、kubernetes 集群已经可用。

由于使用 kubernetes v1.13.1，因此修改 CronJob 的版本 batch/v1beta1 为 batch/v1，Deployment 的版本 extensions/v1beta1 为 apps/v1


#### 部署备份
##### 创建 RBAC 并且部署 etcd operator
1、使用 --namespace=kube-system 参数或者修改脚本中默认的 namespace 为 kube-system，然后在 kubernetes master 节点上，执行如下命令：
```angular2html
bash ./rbac/create_role.sh
```

2、部署 etcd operator
```angular2html
kubectl create -f etcd-operator.yaml
```

##### 部署 etcd backup operator
1、生成 etcd-backup-operator.yaml 中需要使用的各证书的 kubernetes secret（如果需要修改 secret 的名字，则 yaml 文件中相应名称也要修改为一致的）
member.peerSecret 生成方式如下：
```angular2html
kubectl create secret generic etcd-peer-tls --from-file=peer-ca.crt --from-file=peer.crt --from-file=peer.key
```
member.serverSecret 生成方式如下：
```angular2html
kubectl create secret generic etcd-server-tls --from-file=server-ca.crt --from-file=server.crt --from-file=server.key
```
operatorSecret 生成方式如下：
```angular2html
kubectl create secret generic etcd-client-tls --from-file=etcd-client-ca.crt --from-file=etcd-client.crt --from-file=etcd-client.key
```

2、部署 etcd backup operator
```angular2html
kubectl create -f ./backup/etcd-backup-operator.yaml
```
3、生成 AWS Secret

创建一个包含 aws config/credential（对存储数据的 S3 有操作权限）的 kubernetes Secret，备份数据到 S3 时需要用到。

确保如下配置存在且正确：
```angular2html
$ cat $AWS_DIR/credentials
[default]
aws_access_key_id = XXX
aws_secret_access_key = XXX

$ cat $AWS_DIR/config
[default]
region = XXX
```

生成 secret aws
```angular2html
kubectl create secret generic aws --from-file=$AWS_DIR/credentials --from-file=$AWS_DIR/config
```

4 和 5 选择其一

4、使用 Kubernetes CronJob 来备份 etcd 数据

1）修改 configmap.yaml 文件中 S3 的路径（为 S3 桶的全路径）和 etcd 的 endpoints（为 yaml 文件的数组格式），然后创建 ConfigMap 
```angular2html
kubectl create -f ./backup/configmap.yaml
```

2、修改 cronjob.yaml 的 schedule 表达式，默认为：每 30 分钟备份一次，然后创建 CronJob
```angular2html
kubectl create -f ./backup/cronjob.yaml
```
默认为每 30 分钟同步一次，见 schedule 字段指定的表达式，可根据需要按照 Linux crontab 规则自行修改。

5、如果只是备份一次，修改 backup_cr.yaml 中如下字段的值
```angular2html
sed -e 's|<full-s3-path>|mybucket/etcd.backup|g' \
    -e 's|<aws-secret>|aws|g' \
    -e 's|<etcd-cluster-endpoints>|"http://example.etcd.com:2379"|g' \
    ./backup/backup_cr.yaml \
    | kubectl create -f -
```
检查自定义资源 EtcdBackup 的状态
```angular2html
kubectl get EtcdBackup etcd-cluster-backup -o yaml
```

如果要删除该资源，使用命令为：
```angular2html
kubectl delete EtcdBackup etcd-cluster-backup
kubectl delete -f ./backup/etcd-backup-operator.yaml
```

#### 参考资料
1、coreos 官方文档 TLS 配置

https://github.com/coreos/etcd-operator/blob/master/doc/user/cluster_tls.md

2、coreos 官方文档 etcd-backup-operator 部署

https://coreos.com/operators/etcd/docs/latest/user/walkthrough/backup-operator.html


