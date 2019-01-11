### 证书类型介绍
* client certificate 用于通过服务器验证客户端，例如etcdctl，etcd proxy，fleetctl 或 docker 客户端
* server certificate 由服务器使用，并由客户端验证服务器身份，例如 docker 服务器或 kube-apiserver。
* peer certificate 由 etcd 集群成员使用，供它们之间通信使用。


### 生成证书
1、配置 CA 选项
```angular2html
cat << EOF > ca-config.json
{
    "signing": {
        "default": {
            "expiry": "43800h"
        },
        "profiles": {
            "server": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
$ cat << EOF > ca-csr.json
{
    "CN": "My own CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "US",
            "L": "CA",
            "O": "My Company Name",
            "ST": "San Francisco",
            "OU": "Org Unit 1",
            "OU": "Org Unit 2"
        }
    ]
}
```
生成 CA 证书
```angular2html
-bash-4.2# cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
2019/01/08 07:15:50 [INFO] generating a new CA key and certificate from CSR
2019/01/08 07:15:50 [INFO] generate received request
2019/01/08 07:15:50 [INFO] received CSR
2019/01/08 07:15:50 [INFO] generating key: rsa-2048
2019/01/08 07:15:50 [INFO] encoded CSR
2019/01/08 07:15:50 [INFO] signed certificate with serial number 75605667555392511044462158733098500435659887355
-bash-4.2# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  client.json  member1.json  server.json
```


生成服务器端证书


cat server.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server -hostname="172.27.126.226,127.0.0.1,server" - | cfssljson -bare server

```angular2html
-bash-4.2# cat server.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server -hostname="172.27.126.226,127.0.0.1,server" - | cfssljson -bare server
2019/01/08 07:27:04 [INFO] generate received request
2019/01/08 07:27:04 [INFO] received CSR
2019/01/08 07:27:04 [INFO] generating key: ecdsa-256
2019/01/08 07:27:04 [INFO] encoded CSR
2019/01/08 07:27:04 [INFO] signed certificate with serial number 701224914043371800723211735116174875911643048922
-bash-4.2# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  client.json  etcd0.json  server.csr  server.json  server-key.pem  server.pem

```

生成对等证书
```angular2html

cat etcd0.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer -hostname="172.27.126.226,127.0.0.1,server,etcd0" - | cfssljson -bare etcd0

-bash-4.2# cat etcd0.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer -hostname="172.27.126.226,127.0.0.1,server,etcd0" - | cfssljson -bare etcd0
2019/01/08 07:33:13 [INFO] generate received request
2019/01/08 07:33:13 [INFO] received CSR
2019/01/08 07:33:13 [INFO] generating key: ecdsa-256
2019/01/08 07:33:13 [INFO] encoded CSR
2019/01/08 07:33:13 [INFO] signed certificate with serial number 245731282545698611734226399860924120690428324945

-bash-4.2# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  client.json  etcd0.csr  etcd0.json  etcd0-key.pem  etcd0.pem  server.csr  server.json  server-key.pem  server.pem

```

生成客户端证书
```angular2html
cat client.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client - | cfssljson -bare client

-bash-4.2# cat client.json | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client - | cfssljson -bare client
2019/01/08 07:35:35 [INFO] generate received request
2019/01/08 07:35:35 [INFO] received CSR
2019/01/08 07:35:35 [INFO] generating key: ecdsa-256
2019/01/08 07:35:35 [INFO] encoded CSR
2019/01/08 07:35:35 [INFO] signed certificate with serial number 288076749458707937320111751909000301018702098156
2019/01/08 07:35:35 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

```

etcd 容器启动命令为：注意将 http 改为 https
```angular2html
docker run -d \
    --restart=always \
    --name=etcd \
    -p 3379:2379 \
    -p 3380:2380 \
    -v /data/etcd/data:/etcd-data \
    -v /etc/ssl/etcd/:/etc/ssl/etcd \
    gcr.io/etcd-development/etcd:v3.2.25 \
    /usr/local/bin/etcd \
    --name etcd1 \
    --data-dir /etcd-data \
    --listen-client-urls https://0.0.0.0:2379 \
    --advertise-client-urls https://0.0.0.0:2379 \
    --listen-peer-urls https://0.0.0.0:2380 \
    --initial-advertise-peer-urls https://0.0.0.0:2380 \
    --initial-cluster etcd1=https://0.0.0.0:2380 \
    --cert-file=/etc/ssl/etcd/server.pem \
    --key-file=/etc/ssl/etcd/server-key.pem \
    --peer-cert-file=/etc/ssl/etcd/etcd0.pem \
    --peer-key-file=/etc/ssl/etcd/etcd0-key.pem \
    --trusted-ca-file=/etc/ssl/etcd/ca.pem \
    --peer-trusted-ca-file=/etc/ssl/etcd/ca.pem \
    --initial-cluster-token tkn \
    --initial-cluster-state new
```

测试 
```angular2html
curl --cacert /etc/ssl/etcd/ca.pem --cert /etc/ssl/etcd/client.pem --key /etc/ssl/etcd/client-key.pem https://10.93.81.17:2379/health
{"health": "true"}

$ etcdctl --endpoints=[10.93.81.17:2379] --cacert=/etc/ssl/etcd/ca.pem --cert=/etc/ssl/etcd/client.pem --key=/etc/ssl/etcd/client-key.pem member list
     
$ etcdctl --endpoints=[10.93.81.17:2379] --cacert=/etc/ssl/etcd/ca.pem --cert=/etc/ssl/etcd/client.pem --key=/etc/ssl/etcd/client-key.pem put /foo/bar  "hello world"
     
$ etcdctl --endpoints=[10.93.81.17:2379] --cacert=/etc/ssl/etcd/ca.pem --cert=/etc/ssl/etcd/client.pem --key=/etc/ssl/etcd/client-key.pem get /foo/bar

```

```angular2html
-bash-4.2# curl --cacert /etc/ssl/etcd/ca.pem --cert /etc/ssl/etcd/client.pem --key /etc/ssl/etcd/client-key.pem https://172.27.126.226:3379/health
{"health": "true"}
```


```angular2html
curl --cacert /etc/ssl/etcd/ca.pem --cert /etc/ssl/etcd/client.pem --key /etc/ssl/etcd/client-key.pem https://172.27.126.226:3379/v2/keys/xdhuxc-key -XPUT -d value="xdhuxc-value"
```
curl http://10.10.24.28:2379/v2/keys/xdhuxc-key -XPUT -d value="xdhuxc-value"

```angular2html
-bash-4.2# curl --cacert /etc/ssl/etcd/ca.pem --cert /etc/ssl/etcd/client.pem --key /etc/ssl/etcd/client-key.pem https://172.27.126.226:3379/v2/keys/xdhuxc-key -XPUT -d value="xdhuxc-value"
{"action":"set","node":{"key":"/xdhuxc-key","value":"xdhuxc-value","modifiedIndex":6,"createdIndex":6}}
```

```angular2html
-bash-4.2# curl --cacert /etc/ssl/etcd/ca.pem --cert /etc/ssl/etcd/client.pem --key /etc/ssl/etcd/client-key.pem https://172.27.126.226:3379/v2/keys/
{"action":"get","node":{"dir":true,"nodes":[{"key":"/xdhuxc-key","value":"xdhuxc-value","modifiedIndex":6,"createdIndex":6}]}}
-bash-4.2# curl http://172.27.126.226:3379/v2/keys
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
```

修改 prometheus 容器启动命令：
```angular2html
docker run -d \
    --privileged=true \
    --restart=always \
    --name=prometheus \
    --network=host \
    -v /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v /etc/ssl/etcd/:/etc/ssl/etcd \
    prom/prometheus
```

```angular2html
docker run -d \
    --user 0:0 \
    --privileged=true \
    --restart=always \
    --name=prometheus \
    --network=host \
    -v /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v /data/prometheus/rules/etcd3_alert.yml:/prometheus/etcd3_alert.yml \
    -v /etc/ssl/etcd:/etc/ssl/etcd \
    prom/prometheus
```

```angular2html
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['127.0.0.1:9090']
  - job_name: 'etcd'
    scheme: 'https'
    tls_config:
      ca_file: '/etc/ssl/etcd/ca.pem'          # 是否能直接放置文件内容？待测试
      cert_file: '/etc/ssl/etcd/client.pem'
      key_file: '/etc/ssl/etcd/client-key.pem'
    static_configs:
    - targets: ['172.27.126.226:3379']
```


建议部署在原生 prometheus 上，原因如下：

1、由于要以绝对路径方式配置证书文件，在原生方式下，直接将证书文件复制到 prometheus 所在集群上，配置到 prometheus.yml 文件中，然后重启 prometheus 即可；

2、如果以 kubernetes 方式部署，需要将证书文件以 volume 形式映射进 prometheus 容器内，而用一般方式将文件映射进去会导致读取证书文件权限错误，强行增加权限又会导致不安全问题；
另外 kubernetes 的 pod 的漂移会导致证书文件在每台工作节点上都需要存放一份；

3、使用证书文件生成 kubernetes 的 Secret，配置到 Prometheus 中。

### 含证书的 etcd 备份至 AWS S3 与恢复

方案：使用 coreos 提供的 etcd backup operator，

#### 前提条件
1、已部署含自签名证书的 etcd 集群，基于原生方式部署


#### 部署备份
1、设置 RBAC 并且部署 etcd operator
 






1、etcd backup operator 是部署在 kubernetes 上的，需要和原生部署的使用了自签名证书的 etcd 集群通信，etcd backup operator 配置如下：
```angular2html
apiVersion: "etcd.database.coreos.com/v1beta2"
kind: "EtcdCluster"
metadata:
  name: "example"
spec:
  size: 3
  TLS:
    static:
      member:
        peerSecret: etcd-peer-tls
        serverSecret: etcd-server-tls
      operatorSecret: etcd-client-tls
```

2、需要使用证书文件生成 Kubernetes 资源 Secret，
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

3、部署 etcd backup operator
编辑如下 etcd-backup-operator.yaml 文件
```angular2html
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: etcd-backup-operator
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: etcd-backup-operator
    spec:
      containers:
      - name: etcd-backup-operator
        image: quay.io/coreos/etcd-operator:v0.9.3
        command:
        - etcd-backup-operator
        env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
```



 
#### 参考资料
1、coreos 官方文档
https://github.com/coreos/etcd-operator/blob/master/doc/user/cluster_tls.md




#### 参考资料

https://www.jianshu.com/p/1043903bc359

https://www.jianshu.com/p/1043903bc359