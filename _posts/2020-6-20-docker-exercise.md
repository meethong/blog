---
layout: post
title: Docker笔记第二天
categories: [Docker]
description: Docker
keywords: Docker Docker
---

Docker容器部署与数据卷持久化

# Docker实战安装

练习docker部署容器 熟悉相关操作

## Docker安装Nginx

```shell
# 1、搜索镜像 search 建议去docker上搜索 可以看到帮助文档
# 2、下载镜像
# 3、运行测试
[root@localhost ~]# docker pull nginx 
#-d 后台运行
#--name 给容器命名
#-p 宿主机端口:容器内部端口
[root@localhost ~]# docker run -d --name nginx01 -p:3344:80 nginx
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
322d937e89a9        nginx               "/docker-entrypoint.…"   10 seconds ago      Up 4 seconds        0.0.0.0:3344->80/tcp   nginx01
[root@localhost ~]# curl  localhost:3344
#进入容器
[root@localhost ~]# docker exec -it nginx01 /bin/bash
root@322d937e89a9:/# whereis nginx        
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@322d937e89a9:/# cd /etc/nginx/
root@322d937e89a9:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
```

## Docker安装Tomcat

 ```shell
#docker run -it --rm 一般用来测试 用完就删除
docker run -it --rm tomcat 
#先下载Docker
docker pull tomcat
#启动运行
docker run -d  --name tomcat01 -p 80:8080 tomcat
#默认404 因为默认tomcat容器中webapps下没有内容需要把webapps.dist下的内容复制到webapps
[root@localhost ~]# docker exec -it tomcat01 /bin/bash
root@8b2eb48c88d7:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@8b2eb48c88d7:/usr/local/tomcat# cp -r webapps.dist/* webapps/
#刷新再访问即可
 ```

## Docker部署es

```shell
#es 暴露的端口很多
#es 十分耗内存
#es的数据一般需要放到安全目录 挂载
# --net somenetwork 网络配置

[root@localhost ~]# docker run -d --name es  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch
28c62430ef6170fa965d1f3d30f94d71c4fb166b12f18184f35e1d924b77b009
[root@localhost ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                            NAMES
28c62430ef61        elasticsearch       "/docker-entrypoint.…"   21 seconds ago      Up 20 seconds       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es
#测试访问
[root@localhost ~]# curl localhost:9200
{
  "name" : "2Cl8W2u",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "M2RZefmIQH-9xmVofqzbfA",
  "version" : {
    "number" : "5.6.12",
    "build_hash" : "cfe3d9f",
    "build_date" : "2018-09-10T20:12:43.732Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
[root@localhost ~]# 
#查看dokcercpu状态
[root@localhost ~]# docker stats es

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
28c62430ef61        es                  100.69%             1.309GiB / 3.686GiB   35.51%              2.2kB / 0B          0B / 0B             4

#es修改JAVA内存
[root@localhost ~]# docker run -d --name es1  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xms512m" elasticsearch
adf66348505a744a1b2193870e184bdcfe39e3bfb7b32b9d9a77d9ee4cc3897e
#查看dokcercpu状态

```

# 使用数据卷

数据卷可以在容器之间共享和重用
对数据卷的修改会立马生效
对数据卷的更新， 不会影响镜像
数据卷默认会一直存在， 即使容器被删除  

```shell
docker run -it -v  #宿主机路径:容器路径 镜像名
docker run -it -v  /opt/docker-centos/:/home centos /bin/bash
```

## 安装MySQL

```shell
#获取mysql镜像
[root@localhost docker-centos]# docker pull mysql:5.7
#官方测试 docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
#使用数据卷部署
# -d 后台运行
# -p 端口映射
# -v 卷挂载
# --name 容器名字
[root@localhost /]# docker run -d -p 3301:3306 -v /opt/docker-mysql/conf:/etc/mysql/conf.d -v /opt/docker-mysql/data:/var/lib/mysql -eMYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
#启动成功 本地连接一下或者查看3301端口是否存在
```

## 具名挂载和匿名挂载

```shell
#查看所有 volume 的情况
#匿名挂载
#-v 容器内路径
docker run -d -P --name nginx12 -v /etc/nginx nginx
[root@localhost data]# docker volume ls
DRIVER              VOLUME NAME
local               280bc04bb4286c3f63ce7ec6146cb4e3165b04950ab0bf57887fc51e655e6f99
#具名挂载
[root@localhost data]# docker run -d -P --name nginx13 -v nginx:/etc/nginx nginx
9694d2d47e9ff1ee8ca4e020ff6a3f045ac4a0f6d50965ccc3341e556d9903b9
[root@localhost data]# docker volume ls
DRIVER              VOLUME NAME
local               nginx

#查看挂载路径
```

![image-20200620220843507](https://i.opsta.cn/docker/docker-volumes-nginx.png))

所有docker容器内的卷,没有指定目录的情况下都是在/var/lib/docker/volumes/***/_data

可以通过具名挂载方便找到路径

```shell
-v 容器内路径 #匿名挂载
-v /宿主机路径:容器内路径 #指定路径挂载
-v 卷名:容器内路径 #具名挂载
```

扩展

```shell
#可以通过 -v 容器内路径 ro rw 改变读写权限
ro readonly #只读
rw readwrite #读写
一旦设置了ro那么容器就无法写文件 只能通过宿主机来操作
```

# 数据卷容器  

如果你有一些持续更新的数据需要在容器之间共享， 最好创建数据卷容器。
数据卷容器， 其实就是一个正常的容器， 专门用来提供数据卷供其它容器挂载的。
首先， 创建一个名为 dbdata 的数据卷容器：

```shell
[root@localhost ~]#  docker run -d -v /opt/docker/docker-dbdata  --name dbdata training/postgres
```

然后， 在其他容器中使用 --volumes-from 来挂载 dbdata 容器中的数据卷。

```shell
[root@localhost ~]#  docker run -d --volumes-from dbdata --name db1 training/postgres  
```

```shell
[root@localhost ~]#  docker run -d --volumes-from dbdata --name db2 training/postgres
```

可以使用超过一个的 --volumes-from 参数来指定从多个容器挂载不同的数据卷。 也可以从其他已经挂载了数据卷的容器来级联挂载数据卷 。