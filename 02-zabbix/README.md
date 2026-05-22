# Zabbix 监控补充模块

## 说明

本目录记录 Zabbix 与 VictoriaMetrics 联动的实践配置。
通过 Zabbix HTTP 代理监控项主动拉取 VictoriaMetrics 数据，
JavaScript 预处理格式化输出，在 Global View 仪表盘展示
防火墙 / 全网交换机 / 存储交换机 Top5 出口流量排行。

## Dashboard 预览

![Zabbix Global View](docs/zabbix-global-view.png)

## 配置文档

- [Top5 流量 Widget 配置](configs/top5-traffic-widget.md)
