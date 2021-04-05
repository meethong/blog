---
layout: post
title: Prometheus搭建
categories: Prometheus
description: Prometheus
keywords: Prometheus
---



Prometheus下载地址https://prometheus.io/download/

```shell
tar -xvf prometheus-2.25.1.linux-amd64.tar.gz
cd prometheus-2.12.0.linux-amd64
vim /etc/systemd/system/prometheus.service

[Unit]
Description=prometheus
After=network.target
[Service]
Type=simple
User=root
ExecStart=/data/prometheus/prometheus/prometheus --config.file=/data/prometheus/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus/storage
Restart=on-failure
[Install]
WantedBy=multi-user.target

systemctl start prometheus   #启动prometheus
systemctl status prometheus  #查看prometheus运行状态
```

grafana下载地址https://grafana.com/grafana/download

```shell
rpm -ivh grafana-7.4.3-1.x86_64.rpm 
 sudo /bin/systemctl daemon-reload
 /bin/systemctl enable grafana-server.service
 /bin/systemctl start grafana-server.service
 
 访问http://IP:3000/ 默认账号为admin/admin 第一次登录需要修改密码
```

