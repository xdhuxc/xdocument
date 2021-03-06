### 部署

#### 原生方式部署
##### 下载安装包
```angularjs

wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.5.0.linux-amd64.tar.gz

wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz

wget https://github.com/prometheus/alertmanager/releases/download/v0.15.3/alertmanager-0.15.3.linux-amd64.tar.gz

```

##### 启动服务
1、 启动 prometheus

prometheus.yml
```angular2
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['192.168.33.10:9090']
```
启动 prometheus，默认会加载当前路径下的 prometheus.yaml 文件：

```angular2
nohup ./prometheus >> prometheus.log &
```
或者以 Linux 系统服务方式启动
```angular2

```

然后访问如下地址：
```angular2
http:192.168.33.10:9090
```

2、启动 node_exporter
```angular2
ohup ./node_exporter >> node_exporter.log &
```
然后访问如下地址：
```angular2
http://192.168.33.10:9100 
```
或
```angular2
http://192.168.33.10:9100/metrics
```
在 prometheus.yml 中加入如下配置，重启 prometheus
```angular2
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['192.168.33.10:9100']
```

3、部署 grafana

安装 grafana
```angular2
sudo yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.1.4-1.x86_64.rpm
```

启动 grafana 服务
```angular2
systemctl start grafana-server
```

然后访问如下地址：
```angular2
http://192.168.33.10:3000 
```
以 admin/admin 登录进去

然后添加数据源，将 prometheus 相关信息填写进去。

4、部署 clickhouse


#### 容器方式部署
1、启动 prometheus 容器
```
docker run -d --restart=always --name=prometheus --network=host -v /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```
为了便于和其他容器通信，使用 host 网络模式启动 prometheus 容器。

2、启动 node_exporter 容器
```angularjs
docker run -d -p 9100:9100 prom/node-exporter
```
将如下连接信息写入 prometheus.yml 中，重启 prometheus 容器
```angularjs
scrape_configs:
  - job_name: 'node_exporter'
  static_configs:
  - targets: ['192.168.33.10:9100']
- job:
  static_configs:
```

3、启动 grafana 容器
```angularjs
docker run -d --restart=always --name=grafana -p 3000:3000 grafana/grafana
```

4、启动 clickhouse 容器
```angularjs
# 启动 clickhouse-server
docker run -d -p 8123:8123 -p 9000:9000 -p 9009:9009 --name clickhouse-server --ulimit nofile=262144:262144 yandex/clickhouse-server
```

5、启动 clickhouse_exporter 容器，需要指定 clickhouse 的地址
```angularjs
docker run -d -p 9116:9116 f1yegor/clickhouse-exporter -scrape_uri=http://192.168.33.10:8123/
```

6、以 admin/admin 登录进 grafana，导入 clickhouse 的 dashboard 文件，文件地址为：https://grafana.com/dashboards/882
