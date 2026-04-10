# VictoriaMetrics

## 概述

单机版时序数据库，替代 Prometheus 作为长期存储后端。适用场景：监控设备数量多、数据点密集，Prometheus 内存压力大；VictoriaMetrics 相同数据量内存占用约为 Prometheus 的 1/7，且原生支持 Prometheus Remote Write 协议，Telegraf 和 Grafana 无需额外适配。

---

## 目录结构

```
/opt/victoria/
└── victoria-metrics-prod       # 二进制文件，单文件无依赖

/data/victoria/
└── data/                       # 数据存储目录（建议挂载独立数据盘）
    ├── cache/
    ├── data/
    ├── indexdb/
    ├── metadata/
    ├── snapshots/
    └── tmp/
```

---

## 部署步骤

### 1. 下载

```bash
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.110.0/victoria-metrics-linux-amd64-v1.110.0.tar.gz
tar xzf victoria-metrics-linux-amd64-v1.110.0.tar.gz -C /opt/victoria/
```

### 2. 创建数据目录

```bash
mkdir -p /data/victoria/data
```

### 3. 启动测试

```bash
/opt/victoria/victoria-metrics-prod \
  -storageDataPath=/data/victoria/data \
  -httpListenAddr=:8428
```

启动成功输出：

```
started VictoriaMetrics in 0.036 seconds
started server at http://0.0.0.0:8428/
```

### 4. 配置 systemd 服务

```bash
cat <<EOF > /etc/systemd/system/victoria-metrics.service
[Unit]
Description=VictoriaMetrics TSDB
After=network.target

[Service]
Type=simple
User=root
ExecStart=/opt/victoria/victoria-metrics-prod -storageDataPath=/data/victoria/data -httpListenAddr=:8428
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

### 5. 启动

```bash
systemctl daemon-reload && systemctl enable --now victoria-metrics && systemctl status victoria-metrics
```

服务正常运行状态：

```
● victoria-metrics.service - VictoriaMetrics TSDB
   Loaded: loaded (/etc/systemd/system/victoria-metrics.service; enabled)
   Active: active (running) since Mon 2026-01-26 12:47:33 CST; 2 months 12 days ago
 Main PID: 16531 (victoria-metric)
```

---

## 说明

- 单机版无配置文件，所有参数通过 `ExecStart` 启动参数传入
- 默认端口 `8428`，Telegraf outputs.http 和 Grafana 数据源均指向此地址
- Grafana 数据源类型选 `Prometheus`，URL 填 `http://<本机IP>:8428`
- 数据目录建议挂载独立数据盘，避免占用系统盘
