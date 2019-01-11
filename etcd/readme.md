### 基于 Prometheus 和 grafana 监控 etcd

在 prometheus.yml 文件中配置如下内容：
```angular2html
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['127.0.0.1:9090']
  - job_name: 'etcd'
    static_configs:
    - targets: ['127.0.0.1:3379']
```

在 grafana 中导入 grafana.json 文件，文件地址为：
https://coreos.com/etcd/docs/latest/op-guide/grafana.json

注意，修改 datasource 的值为 prometheus 的数据源名称



### 参考资料

http://etcd.doczh.cn/documentation/op-guide/gateway.html