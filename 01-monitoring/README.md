
# 01-monitoring — Telegraf + VictoriaMetrics + Grafana

## 概述

用 Telegraf SNMP 插件统一采集网络设备流量和 GPU 服务器硬件指标，写入 VictoriaMetrics，Grafana 展示。

核心优势：**单一 telegraf.conf 管理全站所有设备**，用 `device_role` 标签区分角色，Grafana 变量一键切换视角。

---

## 组件版本

| 组件 | 版本 | 说明 |
|---|---|---|
| Telegraf | 1.x | 官方包，`apt` / `yum` 安装 |
| VictoriaMetrics | 单机版 | 二进制部署，`-storageDataPath` 指定数据目录 |
| Grafana | 10.x+ | 数据源配置 VictoriaMetrics 地址即可 |

---

## 部署说明

### 1. VictoriaMetrics

```bash
# 二进制直接启动，数据目录按需修改
/opt/victoria/victoria-metrics-prod \
  -storageDataPath=/data/victoria/data \
  -httpListenAddr=:8428
```

systemd 服务文件：

```ini
[Unit]
Description=VictoriaMetrics TSDB
After=network.target

[Service]
Type=simple
User=root
ExecStart=/opt/victoria/victoria-metrics-prod \
  -storageDataPath=/data/victoria/data \
  -httpListenAddr=:8428
Restart=always

[Install]
WantedBy=multi-user.target
```

### 2. Telegraf

```bash
# 安装后替换配置文件
cp telegraf.conf /etc/telegraf/telegraf.conf

# 使用自定义路径
telegraf --config /opt/telegraf/etc/telegraf/telegraf.conf
```

**注意事项：**

- 所有 IP 地址替换为实际设备管理 IP
- SNMP community 替换为实际值
- 防火墙设备如使用非标准端口（如 1161），需在 agents 中显式指定 `"ip:port"`
- GPU 服务器 iDRAC 默认 community 为 `public`，建议生产环境修改

### 3. Grafana 数据源

添加 Prometheus 类型数据源，URL 填写 `http://<victoriaMetrics-host>:8428`

---

## 采集指标说明

### 网络设备（交换机 / 防火墙）

| 指标 | OID | 说明 |
|---|---|---|
| `ifHCInOctets` | 1.3.6.1.2.1.31.1.1.1.6 | 接口入流量（64位计数器） |
| `ifHCOutOctets` | 1.3.6.1.2.1.31.1.1.1.10 | 接口出流量（64位计数器） |
| `ifInErrors` | 1.3.6.1.2.1.2.2.1.14 | 接口入方向错误包 |
| `ifOutErrors` | 1.3.6.1.2.1.2.2.1.20 | 接口出方向错误包 |

> 使用 `ifHCInOctets` 而非 `ifInOctets`，原因：HCIn 是 64 位计数器，万兆口不会发生计数器翻转

### GPU 服务器（Dell XE9680 iDRAC）

| 指标 | 说明 |
|---|---|
| `gpu_temp_1` ~ `gpu_temp_8` | 8 张 H200 GPU 温度（经 scale × 0.1 还原为摄氏度） |

> Dell iDRAC SNMP OID 返回的温度值为实际值 × 10（如 260 = 26.0°C），需配合 `[[processors.scale]]` 还原

---

## device_role 标签说明

telegraf.conf 中每个 `[[inputs.snmp]]` 块都打了 `device_role` 标签，Grafana 中可用该标签做变量过滤：

| device_role | 设备类型 |
|---|---|
| `storage_sw` | 存储交换机 |
| `biz_access_sw` | 业务接入交换机 |
| `core_sw` | 核心交换机 |
| `firewall` | 防火墙 |
| `compute_sw` | 计算网交换机 |
| `oob_access` | 带外管理接入交换机 |
| `GPU-Server-XE9680` | GPU 服务器 iDRAC |

---

## Grafana 参考 PromQL

**按设备角色查看接口流量（bps）：**

```promql
# 入流量 bps（按 agent_host 聚合，过滤角色）
rate(interface_traffic_ifHCInOctets{device_role="$device_role"}[5m]) * 8
```

**GPU 温度（单台服务器 8 卡）：**

```promql
snmp_gpu_temp_1{agent_host="$gpu_host"}
```

---

<!-- ## 截图  -->

<!-- 截图说明：已打码处理，隐藏实际 IP 及设备数量 -->
