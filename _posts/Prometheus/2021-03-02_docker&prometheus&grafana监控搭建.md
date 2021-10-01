---
layout: post
title: 监控系统Prometheus&grafana
categories: [http, https, 运维, 监控]
description: docker安装监控系统Prometheus&grafana，实现Mysql监控（2021-07-28）
keywords: keyword1, keyword2
---



### 1、说明文档

```
https://www.kancloud.cn/willseecloud/prometheus/1904300
```

### 2、本地映射

```
vi /etc/hosts

# 因为局域网ip总是变动，应对容器构建成功后IP改变
{your ip} myhost
```


### 3、prometheus容器启动命令

```
docker run -d -p 9090:9090 -v /Users/mawl/Workspace/deploy/prom/config/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

- prometheus 配置文件

```yaml
#
# my global config
# file name: prometheus.yml
#
#
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

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
    - targets: ['myhost:9090']

  - job_name: 'myhost::mysql-exporter-01'
    static_configs:
    - targets: ['myhost:9104']
  - job_name: 'myhost::grafana-01'
    static_configs:
    - targets: ['myhost:3000']
```


### 4、mysql_exporter容器启动命令

```
docker run -d --name mysql_exporter --restart always -p 9104:9104 -e DATA_SOURCE_NAME="root:123456@(myhost:3306)/" prom/mysqld-exporter
```


### 5、Grafana容器启动命令

```
docker run -d -p 3000:3000 --name=grafana grafana/grafana
```

- Grafana的存储库仪表板地址

```
https://grafana.com/grafana/dashboards/7362
```