---
layout: post
title: Keepalived-2.0.20编译安装
categories: Keepalived
description: Keepalived
keywords: Keepalived对
---

对keeplived不是很熟，简单记录下安装，配置建议参考官网,这几天尝试测试双机部署keepalived+nginx实现高可用，还挺好用的这里就记录下安装吧。

## Keepalived安装

用到的安装包

[https://www.keepalived.org/software/keepalived-2.0.20.tar.gz](https://www.keepalived.org/software/keepalived-2.0.20.tar.gz)

### 编译安装

```
tar -xvf keepalived-2.0.20.tar.gz
cd keepalived-2.0.20
./configure --prefix=/opt/meethong/keepalived
make
make install
```

### 启停服务

```
opt/eetrust/keepalived/sbin/keepalived -D  启动
pkill keepalived 停止
```

### 配置服务

[https://www.keepalived.org/doc/configuration_synopsis.html](https://www.keepalived.org/doc/configuration_synopsis.html)