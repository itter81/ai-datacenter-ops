①grafana：
1、下载：
[root@zabbix-andl grafana]# wget https://dl.grafana.com/oss/release/grafana-10.4.2.linux-amd64.tar.gz
[root@zabbix-andl grafana]# pwd
/opt/grafana
[root@zabbix-andl grafana]# ls  -l 
total 72
drwxr-xr-x  2 root root    62 Apr 10  2024 bin
drwxr-xr-x  3 root root   107 Apr 10  2024 conf
-rw-r--r--  1 root root  5529 Apr 10  2024 Dockerfile
drwxr-xr-x  3 root root    21 Jan 27 10:07 docs
drwxr-xr-x  2 root root     6 Jan 27 10:07 grafana-v10.4.2
-rw-r--r--  1 root root 34523 Apr 10  2024 LICENSE
-rw-r--r--  1 root root   105 Apr 10  2024 NOTICE.md
drwxr-xr-x  2 root root   293 Apr 10  2024 npm-artifacts
drwxr-xr-x  6 root root    58 Jan 27 10:07 packaging
drwxr-xr-x  3 root root    78 Apr 10  2024 plugins-bundled
drwxr-xr-x 16 root root   286 Apr 10  2024 public
-rw-r--r--  1 root root  3261 Apr 10  2024 README.md
drwxr-xr-x  7 root root 12288 Apr 10  2024 storybook
drwxr-xr-x  2 root root    26 Jan 27 10:07 tools
-rw-r--r--  1 root root     8 Apr 10  2024 VERSION
[root@zabbix-andl grafana]# 


2、复制下模板：
[root@zabbix-andl grafana]#  cp /opt/grafana/conf/sample.ini /opt/grafana/conf/grafana.ini


3、修改：
[root@zabbix-andl grafana]# vi  /opt/grafana/conf/grafana.ini
allow_embedding = true
[auth.anonymous]
# enable anonymous access
enabled = true

# specify organization name that should be used for unauthenticated users
org_name = Main Org.

# specify role for unauthenticated users
org_role = Viewer

cookie_samesite = lax



[server]
# Protocol (http, https, h2, socket)
;protocol = http

enable_gzip = true
allow_embedding = true
cross_origin_policy = *


4、启动文件：
[root@zabbix-andl ~]# cat /etc/systemd/system/grafana.service
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
[root@zabbix-andl ~]# 



5、重启：
[root@zabbix-andl grafana]# systemctl daemon-reload  ; systemctl enable grafana --now

