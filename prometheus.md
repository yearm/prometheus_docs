## prometheus

1. 下载 prometheus
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.11.1/prometheus-2.11.1.linux-amd64.tar.gz
```
2. 配置 prometheus.yml
```yml
global:
  scrape_interval:     15s # 拉取 targets 的默认时间间隔
  evaluation_interval: 15s # 执行 rules 的时间间隔
  scrape_timeout:      10s # 拉取一个 target 的超时时间
  #external_labels:        # 额外的属性，会添加到拉取的数据并存到数据库中
    #monitor: 'codelab-monitor'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "rules.yml"
  # - "rules1.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'api'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    scrape_interval: 5s # 拉取 targets 的时间间隔(覆盖全局默认 scrape_interval)
    static_configs:
     - targets: ['localhost:8090','localhost:8091']

  - job_name: 'mysqld'
    # scrape_interval: 8s
    static_configs:
      - targets: ['127.0.0.1:9104']

```

3. 启动 prometheus

```shell
./prometheus --config.file=prometheus.yml
```

## alertmanager

1. 下载 alertmanager

```shell
wget https://github.com/prometheus/alertmanager/releases/download/v0.18.0/alertmanager-0.18.0.linux-amd64.tar.gz
```

2. 配置 alertmanager.yml

```yml
global:
  resolve_timeout: 5m # 默认持续5分钟未接收到告警后标记告警状态为resolved

route:
  group_by: ['alertname']
  group_wait: 10s # 组报警等待时间
  group_interval: 10s # 组报警间隔时间
  repeat_interval: 1h # 重复报警间隔时间
  receiver: 'webhook'
receivers:
- name: 'webhook'
  webhook_configs:
  - url: 'http://localhost:8060/webhook'
    send_resolved: true
```

3. 启动 alertmanager

```shell
./alertmanager --config.file=alertmanager.yml
```

4. webhook 对接钉钉机器人

```shell
git clone https://github.com/yearm/alertmanager_dingtalk_webhook
./cmd/webhook -dingtalkRobot="https://oapi.dingtalk.com/robot/send?access_token=xxx"
```
