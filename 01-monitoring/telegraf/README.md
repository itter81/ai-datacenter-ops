# Telegraf

## 概述

采集层，通过 SNMP 插件轮询网络设备和 GPU 服务器 iDRAC，将指标以 Prometheus Remote Write 格式写入 VictoriaMetrics。

---

## 目录结构

```
/opt/telegraf/
├── etc/
│   └── telegraf/
│       └── telegraf.conf    # 配置文件，见同级目录 telegraf.conf
├── usr/
│   └── bin/
│       └── telegraf         # 二进制文件
└── var/
```

---

## 部署步骤

### 1. 下载

```bash
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.32.3_linux_amd64.tar.gz
tar xzf telegraf-1.32.3_linux_amd64.tar.gz -C /opt/telegraf/ --strip-components=1
```

### 2. 配置文件

配置文件见同级目录 [telegraf.conf](./telegraf.conf)，关键说明：

- `[[outputs.http]]` 写入地址指向 VictoriaMetrics `http://127.0.0.1:8428/api/v1/import/prometheus`
- 每个 `[[inputs.snmp]]` 块通过 `[inputs.snmp.tags]` 打 `device_role` 标签区分设备角色
- GPU 服务器块配合 `[[processors.scale]]` 将 iDRAC 返回温度值 ×0.1 还原为摄氏度
- 防火墙 SNMP 端口非标准（1161），agents 中需显式指定 `ip:1161`

### 3. 配置 systemd 服务

```bash
cat <<EOF > /etc/systemd/system/telegraf.service
[Unit]
Description=Telegraf Agent
After=network.target

[Service]
Type=simple
ExecStart=/opt/telegraf/usr/bin/telegraf --config /opt/telegraf/etc/telegraf/telegraf.conf
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF
```

### 4. 启动

```bash
systemctl daemon-reload && systemctl enable --now telegraf && systemctl status telegraf
```

---

## 注意事项

- 确认目标设备已开启 SNMP v2，community 与配置文件一致
- 设备数量多时适当调大 `metric_buffer_limit`，默认 10000
- Telegraf 无需数据目录，日志默认输出到 systemd journal，用 `journalctl -u telegraf -f` 查看
