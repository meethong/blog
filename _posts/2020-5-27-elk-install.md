---
layout: post
title: 安装ELK日志分析系统
categories: [Elasticsearch,Logstash,Kibana]
description: Elasticsearch Logstash Kibana
keywords: Elasticsearch,Logstash,Kibana
---

elasticsearch 7.7安装,走了不少弯路,还好自己记录了一下,配置不对很容易出问题！！！

## 安装

```shell
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
配置yum源
[root@instance-3 ~]# more /etc/yum.repos.d/log.repo 
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

```shell
yum install logstash #安装logstash

rpm -ivh https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-x86_64.rpm  #安装elasticsearch
```

```shell
rpm -ivh https://artifacts.elastic.co/downloads/kibana/kibana-7.7.0-x86_64.rpm #安装kibana
```

 安装完需要配置一下

```
vim /etc/elasticsearch/jvm.options
# 根据实际情况修改 jvm内存
-Xms128m
-Xmx256m
```

### elasticsearch配置

```
vim /etc/elasticsearch/elasticsearch.yml
node.data : true
network.host: 0.0.0.0
discovery.seed_hosts : ["127.0.0.1"]
cluster.initial_master_nodes : ["instance-3"] #根据自己主机名修改
```

### kibana配置

```
vim /etc/kibana/kibana.yml 
server.host: "0.0.0.0" #此host必须修改外部用户将无法访问
```

## 启停

```
systemctl start elasticsearch 
systemctl start  kibana 
```

## 日志

```
tail -f /var/log/elasticsearch/elasticsearch.log
tail -f /var/log/kibana/kibana.stdout
```

## 参考

[十分钟搭建和使用ELK日志分析系统](https://www.cnblogs.com/fishbook/p/9370089.html)

[官方文档](https://www.elastic.co/guide/index.html)