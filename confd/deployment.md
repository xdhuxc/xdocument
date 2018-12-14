### 安装 confd

1、下载二进制安装文件并复制到 PATH 目录下
```angular2html
# 下载二进制文件
wget https://github.com/kelseyhightower/confd/releases/download/v0.16.0/confd-0.16.0-linux-amd64
# 重命名该文件
mv confd-0.16.0-linux-amd64 confd
# 移动至 PATH 路径下
cp confd /usr/bin/
```

2、创建 confdir
```angular2html
mkdir -p /etc/confd/{conf.d, templates}
```

3、创建 nginx.toml
```angular2html
[template]
prefix = "/myapp"
src = "nginx.tmpl"
dest = "/tmp/nginx.conf"
owner = "nginx"
mode = "0644"
keys = [
  "/subdomain",
  "/upstream",
]
nodes = [
   "http://127.0.0.1:2379"
]
interval = 60
#check_cmd = "/usr/sbin/nginx -t -c {{.src}}"
#reload_cmd = "/usr/sbin/nginx -s reload"
```

4、创建 nginx.tmpl
```angular2html
events {
    use epoll;
    worker_connections  2048;
}

upstream {{getv "/subdomain"}} {
{{range getvs "/upstream/*"}}
    server {{.}};
{{end}}
}

server {
    server_name  {{getv "/subdomain"}}.example.com;
    location / {
        proxy_pass        http://{{getv "/subdomain"}};
        proxy_redirect    off;
        proxy_set_header  Host             $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
   }
}
```

5、部署 etcd 并创建数据
```angular2html
etcdctl --endpoints=http://localhost:2379 set /myapp/subdomain myapp
etcdctl --endpoints=http://localhost:2379 set /myapp/upstream/app2 "10.0.1.100:80"
etcdctl --endpoints=http://localhost:2379 set /myapp/upstream/app1 "10.0.1.101:80"
```

6、启动 confd
```angular2html
confd -onetime -backend etcd -node http://localhost:2379
```

以守护进程方式启动 confd
```angular2html
confd -watch -backend etcd -node http://localhost:2379 &
```

7、创建 DynamoDB 数据表及准备数据
在本地启动 DynamoDB 数据库实例，配置如下环境变量
```angular2html
export AWS_REGION=ap-southeast-1
export DYNAMODB_LOCAL=http://localhost:8000
```
需要在 ~/.aws 目录下创建 credentials 文件，添加任意的符合 AWS 规则的AccessKey，AccessSecret
```angular2html
[default]
aws_access_key_id = ${aws_access_key_id}
aws_secret_access_key = ${aws_access_key_id}
```


创建表 xdhuxc
```angular2html
aws dynamodb create-table \
    --table-name xdhuxc \
    --attribute-definitions AttributeName=key,AttributeType=S \
    --key-schema AttributeName=key,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
    --endpoint-url http://localhost:8000 
```
准备数据
```angular2html
aws dynamodb put-item --table-name xdhuxc \
    --item '{ "key": { "S": "/myapp/subdomain" }, "value": {"S": "myapp"}}' \
    --endpoint-url http://localhost:8000    

aws dynamodb put-item --table-name xdhuxc \
    --item '{ "key": { "S": "/myapp/upstream/app2" }, "value": {"S": "10.0.1.100:80"}}' \
    --endpoint-url http://localhost:8000    

aws dynamodb put-item --table-name xdhuxc \
    --item '{ "key": { "S": "/myapp/upstream/app1" }, "value": {"S": "10.0.1.101:80"}}' \
    --endpoint-url http://localhost:8000  
    
```
插入 JSON 格式数据
```angular2html
aws dynamodb put-item --table-name xdhuxc \
    --item '{ "key": { "S": "/myapp/study/hosts/codelieche" }, "value": {"S": "{\"domain\": \"www.codelieche.com\", \"ip\": \"192.168.1.101\"}" }}' \
    --endpoint-url http://localhost:8000
    
aws dynamodb put-item --table-name xdhuxc \
    --item '{ "key": { "S": "/myapp/study/hosts/codelieche2" }, "value": {"S": "{\"domain\": \"www.codelieche.com\", \"ip\": \"192.168.1.101\"}"}}' \
    --endpoint-url http://localhost:8000
```

dynamodb.tmpl 中加入如下内容：
```angular2html
{{ range gets "/myapp/study/hosts" }}
    {{ $item := json .Value }}
    domain: {{ $item.domain }}
    ip:     {{ $item.ip }}
{{ end }}
```

以守护进程方式启动 confd
```angular2html
confd -onetime -backend dynamodb -node http://localhost:8000 -table xdhuxc --log-level debug
```


终极格式：
dynamodb.toml
```angular2html
[template]
prefix = "/myapp"
src = "dynamodb.tmpl"
dest = "/tmp/prometheus.conf"
owner = "root"
mode = "0644"
keys = [
  "/subdomain",
  "/upstream",
  "/study/hosts"
]
nodes = [
   "http://localhost:8000"
]
table = "xdhuxc"
interval = 60
```

dynamodb.tmpl
```angular2html
events {
    use epoll;
    worker_connections  2048;
}

upstream {{getv "/subdomain"}} {
{{range getvs "/upstream/*"}}
    server {{.}};
{{end}}
}

server {
    server_name  {{getv "/subdomain"}}.example.com;
    location / {
        proxy_pass        http://{{getv "/subdomain"}};
        proxy_redirect    off;
        proxy_set_header  Host             $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
   }
}

{{range gets "/study/hosts/*"}}
{{$data := json .Value}}
domain: {{$data.domain}}
ip: {{$data.ip}}
{{end}}
```



