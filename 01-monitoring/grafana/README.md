## Grafana

### 1. 下载

```bash
wget https://dl.grafana.com/oss/release/grafana-10.4.2.linux-amd64.tar.gz
```

### 2. 复制配置模板

```bash
cp /opt/grafana/conf/sample.ini /opt/grafana/conf/grafana.ini
```

### 3. 修改配置

```bash
vi /opt/grafana/conf/grafana.ini
```

关键配置项：

```ini
allow_embedding = true

[auth.anonymous]
enabled = true
org_name = Main Org.
org_role = Viewer
cookie_samesite = lax

[server]
enable_gzip = true
allow_embedding = true
cross_origin_policy = *
```

### 4. 配置 systemd 服务

```ini
# /etc/systemd/system/grafana.service
[Unit]
Description=Grafana
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/grafana
ExecStart=/opt/grafana/bin/grafana-server --config=/opt/grafana/conf/grafana.ini web
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 5. 启动

```bash
systemctl daemon-reload && systemctl enable grafana --now
```
