### prometheus 配置
```angular2html
# Prometheus 全局配置项
global:
  scrape_interval:     15s # 设置抓取数据的周期，默认为：1min
  evaluation_interval: 15s # 设置更新 rules 文件的周期，默认为：1min
  scrape_timeout:      15s # 设置抓取数据的超时时间，默认为：10s
  external_labels:         # 额外的属性，会添加到拉取的数据并存储到数据库中。
    monitor: 'codelab_monitor' 
  

# Alertmanager 配置
alerting:
  alertmanagers:
  - static_configs:
    - targets: ["localhost:9093"] # 设定 alertmanager 和 prometheus 的交互接口，也就是 alertmanager 监听的 IP 地址和端口。

# rule 配置，首次读取默认加载，之后根据 evaluation_interval 设定的周期加载
rule_files:
  # - "prometheus_rules.yml"
  # - "alertmanager_rules.yml"

# scape 配置，即提供数据的端点的配置
scrape_configs:
  - job_name: 'prometheus'            # job_name 默认以 job=<job_name> 的形式写入时间序列的标签之中，可以用于查询使用
    scrape_interval: 15s              # 抓取周期，默认采用全局配置
    # 指标路径默认为：/metrics 
    # 协议默认为：http 
    static_configs:                   # 静态配置
    - targets: ['192.168.33.10:9090'] # prometheus 抓取数据的地址，即 instance 实例项
    
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['192.168.33.10:9000']    
```
动态更新 prometheus 的配置，即热更新加载，有如下两种方式：
1、向 prometheus 进程发送 SIGHUP 信号。

2、使用如下命令：
```angular2html
curl -X POST http://localhost:9090/-/reload
```

### AlertManager 配置
```angular2html
# Alertmanager 全局配置项
global:
  resolve_timeout: 5m                                    # 处理超时时间，默认为：5min
  smtp_smarthost: 'smtp.163.com:25'                      # 邮箱 SMTP 服务器
  smtp_from: 'abc@163.com'                               # 发件邮箱名称
  smtp_auth_username: 'xdhuxc@163.com'                   # 邮箱名称
  smtp_auth_password: 'Abc@19873'                        # 邮箱密码或授权码
  wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/' # 企业微信地址

# 定义模板
templates:
  - 'template/*.tmpl'
# 定义路由树信息
route:
  group_by: ['group_name']     # 报警分组依据
  group_wait: 10s              # 第一次等待多久时间发送一组警报的通知
  group_internal: 10s          # 下一次发送新警报前的等待时间
  repeat_interval: 1m          # 发送重复警报的周期，对于 email 配置中，此项不可以设置过低，否则将会由于邮件发送太过频繁，被 SMTP 服务器拒绝
  receiver: 'email'            # 发送警报的接收者的名称
  
# 定义警报接收者的信息
receivers:
  - name: 'email'                             # 警报
    email_configs:                            # 邮箱配置
    - to: 'xdhuxc@163.com'                    # 接收警报的 email 配置
      html: '{{ template "email.html" . }}'   # 设置邮件的内容模板
      headers: { Subject: "[WARN] 报警邮件" }  # 接收邮件的标题
      
    webhook_configs:                          # webhook 配置
    - url: 'http://localhost:5001'
    send_resolved: true
    
    wechat_configs:                           # 企业微信报警配置
    - send_resovled: true
      to_party: '1'                           # 接收组的 ID
      agent_id: '1022321'                     # 企业微信——》自定义应用——》AgentID
      corp_id: 'skjsdjgldfg'                  # 企业信息（我的企业——》CorpID【在底部】）
      api_secret: 'sfsdgdfgd'                 # 企业微信——》自定义应用——》Secret
      message: '{{ template "wechat.html" }}' # 发送消息模板的设置

# 覆盖规则
# 一个 inhibit 规则是在与另一组匹配器匹配的警报存在的条件下，使匹配一组匹配器的警报失效的规则，两个警报必须具有一组相同的标签
inhibit_rules:
  - source_match:
      severity: 'critical'
      target_match:
        severity: 'warning'
      equal: ['alertname', 'dev', 'instance']
```

### 在 prometheus 模块中定义报警规则

alertmanager_rules.yml 示例如下：（与 prometheus 同目录下）
```angular2html
groups:
  - name: alertmanager-rules
    rules:
    - alert: InstanceDown         # 报警名称
      expr: up == 0               # 报警的判定条件，参考 prometheus 高级查询来设定
      for: 2m                     # 满足报警条件持续时间多久后，才会发送报警
      labels:
        team: node
      annotations:
        summary: "{{ $labels.instance }}: has been down."
        description: "{{ $labels.instance }}: job {{ $labels.job }} has been down."
        value: {{ $value }}
```

报警信息生命周期的三种状态
* inactive：表示当前报警信息既不是 firing 状态，也不是 pending 状态
* pending：表示在设置的阈值时间范围内被激活的
* firing：表示超过设置的阈值时间被激活的

在 alertmanager 的配置地址，即可查看所监控到的报警信息
```angular2html
http://localhost:9093/#/alerts
```

