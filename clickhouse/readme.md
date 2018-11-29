### 部署 clickhouse_exporter

1、项目地址：https://github.com/f1yegor/clickhouse_exporter

2、下载该项目并编译为二进制文件，移动至 /usr/bin/ 目录下

3、配置如下环境变量
```angular2html
export CLICKHOUSE_USER="default"
export CLICKHOUSE_PASSWORD=""
```

4、以 Linux 系统服务方式启动之
```angular2html
[Unit]
Description=Clickhouse Exporter
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/clickhouse_exporter -scrape_uri=http://192.168.33.10:8123/
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

或者可以基于 docker 容器方式部署
```angular2html
docker run -d -p 9116:9116 f1yegor/clickhouse-exporter -scrape_uri=http://192.168.33.10:8123/
```