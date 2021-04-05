---
layout: post
title: Docker笔记第四天
categories: [Docker]
description: Docker
keywords: Docker 
---

了解 Docker 网络概念

# docker网络

测试 tomcat

```shell
docker run -d -P --name tomcat01 tomcat

[root@localhost /]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
60: eth0@if61: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
failed to resize tty, using default size
```

1. Docker 容器之间的连接使用的桥接 用的是 veth-pair技术
2. 容器不指定网络 默认都是 Docker 路由 默认分配一个地址

```shell
[root@localhost /]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known
#可以使用 --link实现网络通畅 但不推荐 原理就是在hosts文件里写了配置
[root@localhost /]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
```

## 自定义网络

> 查看所有的 docker 网络

![docker_2020-06-25_23-00-38](https://i.opsta.cn/docker/docker_2020-06-25_23-00-38.png)

**网络模式**

- bridge：桥接 docker(默认)
- none：不配置网络
- host:  和宿主机共享网络
- container : 容器网络连通！(用的少 局限性很大)

**测试**

```shell
#docker 创建容器默认用的bridge 即docker0网卡
[root@localhost docker]# docker run -d -P --name tomcat01 --net bridge tomcat

```

## 自定义网络

```shell
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```

![docker_2020-06-25_23-15-56](https://i.opsta.cn/docker/docker_2020-06-25_23-15-56.png)

配置完网络就可以直接ping通了

![docker_2020-06-25_23-17-54](https://i.opsta.cn/docker/docker_2020-06-25_23-17-54.png)

## 网络连通

```shell
#测试把tomcat01 加入mynet网络里
[root@localhost]#  docker network connect mynet tomcat01
#即一个容器两个ip 地址
```

![docker_2020-06-25_23-30-29](https://i.opsta.cn/docker/docker_2020-06-25_23-30-29.png) 